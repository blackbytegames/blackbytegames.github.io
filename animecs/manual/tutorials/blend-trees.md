# Blend Trees

Blend trees smoothly interpolate between multiple animations based on parameters. This tutorial covers all blend tree types Animecs supports.

## Blend Tree Types

| Type | Parameters | Use Case |
|------|-----------|----------|
| 1D | 1 float | Walk/run speed blending |
| 2D Simple Directional | 2 floats | Movement with direction (forward/strafe) |
| 2D Freeform Directional | 2 floats | Same as simple but better blending |
| 2D Freeform Cartesian | 2 floats | Position-based blending |
| Direct | N floats | Manual weight control per clip |

## 1D Blend Tree

Blends based on a single parameter. Most common for locomotion.

### Setup in Animator

1. Create blend tree in state
2. Set blend type: **1D**
3. Add parameter: `Speed`
4. Add motions:
   - Idle (threshold: 0.0)
   - Walk (threshold: 2.0)
   - Run (threshold: 5.0)

### Runtime Control

```csharp
[BurstCompile]
partial struct LocomotionJob : IJobEntity
{
    public EntityManager EntityManager;

    void Execute(Entity entity, in CharacterVelocity velocity)
    {
        float speed = math.length(velocity.Value);
        int speedHash = Animator.StringToHash("Speed");
        
        AnimecsController.SetFloat(EntityManager, entity, speedHash, speed);
    }
}
```

**How it works:**
- Speed 0.0 → 100% Idle
- Speed 1.0 → 50% Idle, 50% Walk
- Speed 2.0 → 100% Walk
- Speed 3.5 → 50% Walk, 50% Run
- Speed 5.0+ → 100% Run

Animecs interpolates between nearest thresholds.

## 2D Blend Trees

Use two parameters for directional movement.

### 2D Simple Directional

Best for character movement (forward/backward + strafe).

**Animator setup:**
1. Blend type: **2D Simple Directional**
2. Parameters: `SpeedX`, `SpeedY`
3. Add motions at positions:
   - Idle (0, 0)
   - Walk Forward (0, 1)
   - Walk Backward (0, -1)
   - Strafe Left (-1, 0)
   - Strafe Right (1, 0)

**Runtime:**

```csharp
[BurstCompile]
partial struct Movement2DJob : IJobEntity
{
    public EntityManager EntityManager;

    void Execute(Entity entity, in CharacterInput input)
    {
        int speedXHash = Animator.StringToHash("SpeedX");
        int speedYHash = Animator.StringToHash("SpeedY");
        
        AnimecsController.SetFloat(EntityManager, entity, speedXHash, input.MoveX);
        AnimecsController.SetFloat(EntityManager, entity, speedYHash, input.MoveY);
    }
}
```

**How it works:**  
Blends based on direction similarity. Input (0.7, 0.7) will blend Walk Forward and Strafe Right.

### 2D Freeform Directional

Same as Simple Directional but with better blending quality. Slightly more expensive.

Use when Simple Directional produces visible pops between animations.

### 2D Freeform Cartesian

Blends based on distance to motion positions. Use when motions represent distinct states rather than directions.

**Example: Aim offset**
- Look Center (0, 0)
- Look Up (0, 1)
- Look Down (0, -1)
- Look Left (-1, 0)
- Look Right (1, 0)

Parameter values directly map to positions.

## Direct Blend Tree

Each motion has its own parameter. Full manual control.

### Setup

1. Blend type: **Direct**
2. Add motions:
   - Walk (parameter: `WalkWeight`)
   - Run (parameter: `RunWeight`)
   - Jump (parameter: `JumpWeight`)

### Runtime

```csharp
int walkWeightHash = Animator.StringToHash("WalkWeight");
int runWeightHash = Animator.StringToHash("RunWeight");

// Blend 30% walk, 70% run
AnimecsController.SetFloat(entityManager, entity, walkWeightHash, 0.3f);
AnimecsController.SetFloat(entityManager, entity, runWeightHash, 0.7f);
```

**Normalization:**  
If total weight > 1.0, Animecs normalizes automatically.

## Complete Example: Character Locomotion

```csharp
[BurstCompile]
public partial struct CharacterLocomotionSystem : ISystem
{
    private static readonly int SpeedXHash = Animator.StringToHash("SpeedX");
    private static readonly int SpeedYHash = Animator.StringToHash("SpeedY");
    private static readonly int JumpHash = Animator.StringToHash("Jump");

    public void OnUpdate(ref SystemState state)
    {
        var entityManager = state.EntityManager;

        foreach (var (input, velocity, entity) in 
            SystemAPI.Query<RefRO<CharacterInput>, RefRO<CharacterVelocity>>()
                     .WithAll<AnimecsTag>()
                     .WithEntityAccess())
        {
            // 2D blend tree for movement
            float3 localVelocity = input.ValueRO.Forward * velocity.ValueRO.Value.z + 
                                   input.ValueRO.Right * velocity.ValueRO.Value.x;

            float speedX = localVelocity.x;
            float speedY = localVelocity.z;

            AnimecsController.SetFloat(entityManager, entity, SpeedXHash, speedX);
            AnimecsController.SetFloat(entityManager, entity, SpeedYHash, speedY);
        }
    }
}
```

## Blend Tree Performance

### Weight Calculation Cost

Blend tree evaluation runs in `AnimecsBlendTreeSystem`. Cost per entity:

| Type | Cost |
|------|------|
| 1D | Low (linear search for thresholds) |
| 2D Directional | Medium (dot products + normalization) |
| 2D Cartesian | Medium (distance calculations) |
| Direct | Low (direct parameter read) |

### Active Motion Limit

Animecs supports up to **8 motions per blend tree**. Only motions with weight > 0.01 are considered active.

For 2D blend trees, typically 2-4 motions are active at once.

### LOD Impact

Blend tree weights update based on LOD:
- **High LOD:** Every frame
- **Medium LOD:** Every 2 frames (cached)
- **Low LOD:** Every 8 frames (cached)

Parameters still update every frame, but weight recalculation respects LOD.

## Debugging Blend Trees

### Check Active Motions

With `AnimecsDebuggerAuthoring` enabled, the overlay shows:
- "BlendTree: 3 active" (number of motions with weight > 0)

### Read Blend Weights

```csharp
var stateData = entityManager.GetComponentData<AnimecsStateData>(entity);
var blendTreeData = stateData.BlendTreeData;

// Weights are stored in Weight0 through Weight7
float firstMotionWeight = blendTreeData.Weight0;
float secondMotionWeight = blendTreeData.Weight1;
```

### Verify Parameter Values

```csharp
float paramX = AnimecsController.GetFloat(entityManager, entity, speedXHash);
float paramY = AnimecsController.GetFloat(entityManager, entity, speedYHash);

Debug.Log($"Blend params: ({paramX}, {paramY})");
```

## Best Practices

### Normalize Input

For 2D directional blend trees, normalize input vectors:

```csharp
float2 input = new float2(moveX, moveY);
if (math.lengthsq(input) > 1f)
{
    input = math.normalize(input);
}

AnimecsController.SetFloat(entityManager, entity, speedXHash, input.x);
AnimecsController.SetFloat(entityManager, entity, speedYHash, input.y);
```

### Place Motions Strategically

For 2D blend trees, motion positions matter:

**Good placement:**
```
        (0, 1)
          |
(-1, 0) - (0, 0) - (1, 0)
          |
        (0, -1)
```

**Bad placement:**
```
(0, 1) - (0.1, 0.9)
         (0.2, 0.8)  // Too close together
```

Spread motions evenly for smooth blending.

### Use Direct Blend for Additive

Direct blend trees work well for additive animations:

```csharp
// Base locomotion in layer 0
// Additive aim in layer 1 (if Animecs supported layers - it doesn't)

// Workaround: Use direct blend tree
AnimecsController.SetFloat(entityManager, entity, baseWalkHash, 1.0f);
AnimecsController.SetFloat(entityManager, entity, aimUpHash, aimAmount);
```

## Common Issues

**Blend looks snappy** - Motion positions too close or thresholds uneven. Spread them out.

**Wrong blend direction** - 2D parameters might be swapped. Verify X/Y mapping.

**Some motions never blend** - Check weight > 0.01 threshold. Very small weights are ignored.

**Blend tree doesn't update** - Verify you're setting the correct parameter hashes.

---

**Next:** [LOD System →](lod-system.md) - Performance optimization through distance-based updates