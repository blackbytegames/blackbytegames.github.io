# AnimecsController

Static API for controlling animations at runtime. All methods are Burst-compatible.

## State Control

### Play (By Name)

```csharp
static void Play(
    EntityCommandBuffer.ParallelWriter ecb,
    int index,
    Entity entity,
    FixedString64Bytes stateName,
    float transitionDuration,
    AnimecsCurveType curveType = AnimecsCurveType.Smooth,
    bool overrideExitTime = false,
    bool resetTime = true)
```

Transitions to target state by name.

**Parameters:**
- `ecb` - Entity command buffer for deferred execution
- `index` - Entity index in query
- `entity` - Target entity
- `stateName` - State name from Animator Controller
- `transitionDuration` - Blend time in seconds (0 = instant)
- `curveType` - Blend interpolation curve
- `overrideExitTime` - Ignore exit time requirement
- `resetTime` - Start target state at time 0

**String Overload:**
```csharp
static void Play(EntityCommandBuffer.ParallelWriter ecb, int index, Entity entity, 
                 string stateName, float transitionDuration, ...)
```

### Play (By Index)

```csharp
static void Play(
    EntityCommandBuffer.ParallelWriter ecb,
    int index,
    Entity entity,
    int stateIndex,
    float transitionDuration,
    AnimecsCurveType curveType = AnimecsCurveType.Smooth,
    bool overrideExitTime = false,
    bool resetTime = true)
```

Transitions to target state by index. Faster than name lookup.

### SetStatePaused

```csharp
static void SetStatePaused(ref AnimecsStateData stateData, bool paused)
```

Pauses or resumes animation time progression. Modify `AnimecsStateData` directly, then write back to entity.

### SetKeyframeOnlyMode

```csharp
static void SetKeyframeOnlyMode(ref AnimecsStateData stateData, bool keyframeOnly)
```

Disables frame interpolation when enabled. Forces exact keyframe sampling.

## Parameter Control

### SetFloat

```csharp
static void SetFloat(EntityManager entityManager, Entity entity, 
                     int parameterHash, float value)
```

Sets float parameter value. Hash from `Animator.StringToHash()`.

### GetFloat

```csharp
static float GetFloat(EntityManager entityManager, Entity entity, int parameterHash)
```

Reads float parameter value. Returns 0 if parameter not found.

### SetInt

```csharp
static void SetInt(EntityManager entityManager, Entity entity, 
                   int parameterHash, int value)
```

Sets int parameter value.

### GetInt

```csharp
static int GetInt(EntityManager entityManager, Entity entity, int parameterHash)
```

Reads int parameter value. Returns 0 if parameter not found.

### SetBool

```csharp
static void SetBool(EntityManager entityManager, Entity entity, 
                    int parameterHash, bool value)
```

Sets bool parameter value.

### GetBool

```csharp
static bool GetBool(EntityManager entityManager, Entity entity, int parameterHash)
```

Reads bool parameter value. Returns false if parameter not found.

### SetTrigger

```csharp
static void SetTrigger(EntityManager entityManager, Entity entity, int parameterHash)
```

Activates trigger parameter. Automatically resets after transition.

### ResetTrigger

```csharp
static void ResetTrigger(EntityManager entityManager, Entity entity, int parameterHash)
```

Manually resets trigger parameter to false.

## State Queries

### GetCurrentStateIndex

```csharp
static int GetCurrentStateIndex(in AnimecsStateData stateData)
```

Returns current state index.

### GetCurrentStateTime

```csharp
static float GetCurrentStateTime(in AnimecsStateData stateData)
```

Returns current animation time in seconds.

### IsTransitioning

```csharp
static bool IsTransitioning(in AnimecsStateData stateData)
```

Checks if entity is blending between states.

### GetTransitionProgress

```csharp
static float GetTransitionProgress(in AnimecsStateData stateData)
```

Returns blend progress (0-1). Returns 0 if not transitioning.

### FindStateIndex

```csharp
static int FindStateIndex(ref AnimecsStateMachineBlobAsset stateMachine, 
                          FixedString64Bytes stateName)
```

Finds state index by name. Returns -1 if not found.

### GetStateName

```csharp
static FixedString64Bytes GetStateName(ref AnimecsStateMachineBlobAsset stateMachine, 
                                        int stateIndex)
```

Returns state name by index. Returns empty string if index invalid.

## LOD Control

### GetLODConfig

```csharp
static AnimecsLODConfig GetLODConfig(EntityManager entityManager)
```

Retrieves global LOD configuration. Returns default config if singleton doesn't exist.

### SetLODConfig

```csharp
static void SetLODConfig(EntityCommandBuffer ecb, AnimecsLODConfig config)
```

Updates global LOD configuration.

### SetLODLevel

```csharp
static void SetLODLevel(EntityManager entityManager, Entity entity, AnimecsLODLevel level)
```

Forces specific LOD level for entity. Disables automatic LOD calculation.

**LOD Levels:**
- `High` - Every frame update
- `Medium` - Every 2 frames (default)
- `Low` - Every 8 frames (default)
- `Off` - No updates

### GetLODLevel

```csharp
static AnimecsLODLevel GetLODLevel(EntityManager entityManager, Entity entity)
```

Returns current LOD level for entity.

## Utility Functions

### HasStateMachineSupport

```csharp
static bool HasStateMachineSupport(EntityManager entityManager, Entity entity)
```

Checks if entity has required Animecs components.

### GetStateCount

```csharp
static int GetStateCount(EntityManager entityManager, Entity entity)
```

Returns total number of states in entity's state machine. Returns 0 if no state machine.

### GetParameterCount

```csharp
static int GetParameterCount(EntityManager entityManager, Entity entity)
```

Returns total number of parameters. Returns 0 if no parameters.

### MapVelocityToStateName

```csharp
static FixedString64Bytes MapVelocityToStateName(
    float3 velocity,
    float walkingThreshold = 0.5f,
    float runningThreshold = 3f)
```

Converts velocity magnitude to locomotion state name ("Idle", "Walking", or "Running").

## Curve Types

```csharp
public enum AnimecsCurveType : byte
{
    Linear,   // Constant blend speed
    Smooth,   // Ease in and out (default)
    EaseIn,   // Slow start, fast end
    EaseOut   // Fast start, slow end
}
```

## Usage Examples

### Basic State Control

```csharp
[BurstCompile]
partial struct PlayerControlJob : IJobEntity
{
    public EntityCommandBuffer.ParallelWriter ECB;

    void Execute([EntityIndexInQuery] int index, Entity entity, in PlayerInput input)
    {
        if (input.JumpPressed)
        {
            AnimecsController.Play(ECB, index, entity, "Jump", 
                transitionDuration: 0.15f,
                curveType: AnimecsCurveType.EaseOut,
                overrideExitTime: true);
        }
    }
}
```

### Parameter-Driven Locomotion

```csharp
var entityManager = state.EntityManager;
int speedHash = Animator.StringToHash("Speed");

foreach (var (velocity, entity) in 
    SystemAPI.Query<RefRO<CharacterVelocity>>()
             .WithAll<AnimecsTag>()
             .WithEntityAccess())
{
    float speed = math.length(velocity.ValueRO.Value);
    AnimecsController.SetFloat(entityManager, entity, speedHash, speed);
}
```

### Reading Animation State

```csharp
var stateData = entityManager.GetComponentData<AnimecsStateData>(entity);

if (AnimecsController.IsTransitioning(stateData))
{
    float progress = AnimecsController.GetTransitionProgress(stateData);
    // Apply visual effect during transition
}

float animTime = AnimecsController.GetCurrentStateTime(stateData);
// Sync gameplay with animation timing
```

### Manual LOD Override

```csharp
// Boss entity always runs at high quality
if (entityManager.HasComponent<BossTag>(entity))
{
    AnimecsController.SetLODLevel(entityManager, entity, AnimecsLODLevel.High);
}
```

---