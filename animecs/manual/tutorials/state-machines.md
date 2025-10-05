# State Machines

State machines define how animations transition based on conditions. This tutorial covers parameter-driven transitions, animation events, and any-state transitions.

## How Transitions Work

Animecs evaluates transitions every frame (respecting LOD). When conditions are met, it automatically blends to the target state.

### Condition Types

Transitions use conditions based on parameter comparisons:

| Mode | Parameter Type | Description |
|------|---------------|-------------|
| `Greater` | Float, Int | Value > threshold |
| `Less` | Float, Int | Value < threshold |
| `Equal` | Float, Int | Value == threshold |
| `NotEqual` | Float, Int | Value != threshold |
| `If` | Bool, Trigger | Value is true |
| `IfNot` | Bool, Trigger | Value is false |

### Exit Time

Transitions can wait for animation completion before firing.

**Has Exit Time = true:**  
State must reach `exitTime` (normalized 0-1) before conditions are checked.

**Has Exit Time = false:**  
Conditions checked immediately.

## Parameter-Driven Transitions

Set up your Animator Controller with conditions, then update parameters at runtime.

### Example: Walk to Run

**In Unity Animator Controller:**
1. Create Float parameter: `Speed`
2. Add transition: Walk → Run
3. Condition: `Speed Greater 3.0`
4. Duration: 0.3s

**At runtime:**

```csharp
[BurstCompile]
partial struct UpdateLocomotionJob : IJobEntity
{
    public EntityManager EntityManager;

    void Execute(Entity entity, in CharacterVelocity velocity)
    {
        float speed = math.length(velocity.Value);
        int speedHash = Animator.StringToHash("Speed");
        
        AnimecsController.SetFloat(EntityManager, entity, speedHash, speed);
        // Transition happens automatically when Speed > 3.0
    }
}
```

### Multiple Conditions

Transitions can have multiple conditions. **All must be true** for transition to fire.

**Animator setup:**
- Idle → Attack
- Condition 1: `Trigger "Attack"`
- Condition 2: `Bool "Grounded" == true`

**Runtime:**

```csharp
int attackHash = Animator.StringToHash("Attack");
int groundedHash = Animator.StringToHash("Grounded");

// Only transitions if both are true
AnimecsController.SetBool(entityManager, entity, groundedHash, true);
AnimecsController.SetTrigger(entityManager, entity, attackHash);
```

## Any-State Transitions

Any-state transitions can fire from **any** current state. Useful for interrupts like damage reactions or death.

### Setup in Animator

1. Right-click state machine background
2. Create "Any State" transition to target
3. Add conditions

### Example: Death Interrupt

**Animator:**
- Any State → Death
- Condition: `Bool "IsDead" == true`
- Duration: 0.0s (instant)

**Runtime:**

```csharp
void OnCharacterDeath(Entity entity, EntityManager entityManager)
{
    int isDeadHash = Animator.StringToHash("IsDead");
    AnimecsController.SetBool(entityManager, entity, isDeadHash, true);
    
    // Death animation plays immediately from any state
}
```

## Animation Events

Fire code when animation reaches specific frames. Useful for footsteps, weapon hits, particle effects.

### Setup in Unity

1. Select animation clip in Project
2. Open Animation window
3. Add event at desired frame
4. Set function name (e.g., "OnFootstep")

Animecs bakes these events into the state blob.

### Detecting Events at Runtime

Events populate the `AnimecsTriggeredEvent` buffer:

```csharp
[BurstCompile]
public partial struct FootstepSystem : ISystem
{
    public void OnUpdate(ref SystemState state)
    {
        foreach (var (eventBuffer, entity) in 
            SystemAPI.Query<DynamicBuffer<AnimecsTriggeredEvent>>()
                     .WithAll<FootstepAudio>()
                     .WithEntityAccess())
        {
            foreach (var evt in eventBuffer)
            {
                if (evt.EventName == "OnFootstep")
                {
                    // Play footstep sound
                    PlayFootstepAudio(evt.FloatParameter); // volume
                }
            }
            
            // Clear events after processing
            eventBuffer.Clear();
        }
    }
}
```

### Event Data

Each `AnimecsTriggeredEvent` contains:

```csharp
public struct AnimecsTriggeredEvent
{
    public FixedString32Bytes EventName;    // Function name
    public byte StateIndex;                 // Which state fired it
    public ushort FrameIndex;               // Frame number
    public float TriggerTime;               // Animation time
    public float FloatParameter;            // Custom data
    public int IntParameter;                // Custom data
}
```

### Event Example: Weapon Attack

**Animation setup:**
1. Add event at frame 15 of "Attack" animation
2. Function name: "OnWeaponHit"
3. Float parameter: 25 (damage)

**Runtime:**

```csharp
[BurstCompile]
partial struct WeaponHitSystem : ISystem
{
    public void OnUpdate(ref SystemState state)
    {
        var ecb = SystemAPI.GetSingleton<BeginSimulationEntityCommandBufferSystem.Singleton>()
            .CreateCommandBuffer(state.WorldUnmanaged);

        foreach (var (eventBuffer, weaponData, entity) in 
            SystemAPI.Query<DynamicBuffer<AnimecsTriggeredEvent>, RefRO<WeaponData>>()
                     .WithEntityAccess())
        {
            foreach (var evt in eventBuffer)
            {
                if (evt.EventName == "OnWeaponHit")
                {
                    float damage = evt.FloatParameter;
                    ApplyDamageInRadius(weaponData.ValueRO.Range, damage, ecb);
                }
            }
            
            eventBuffer.Clear();
        }
    }
}
```

## Transition Best Practices

### Use Triggers for Actions

Triggers reset automatically after use. Perfect for one-shot actions:

```csharp
// Jump, Attack, Reload, etc.
AnimecsController.SetTrigger(entityManager, entity, jumpHash);
```

Don't use for continuous states:

```csharp
// WRONG: Trigger for movement
AnimecsController.SetTrigger(entityManager, entity, walkHash); // Will reset!

// CORRECT: Bool or Float
AnimecsController.SetFloat(entityManager, entity, speedHash, 2.5f);
```

### Exit Time for Attack Chains

When you want full animation completion:

**Animator:**
- Attack1 → Attack2
- Has Exit Time: true
- Exit Time: 0.9 (90% complete)
- Condition: `Trigger "Attack"`

This ensures Attack1 finishes before Attack2 starts.

### Blend Strength

Transition `BlendStrength` in Animator Controller affects interpolation curve:
- 0.0 = Linear
- 0.5 = Smooth (default)
- 1.0 = Very smooth (ease in/out)

Animecs respects this during baking.

## Debugging Transitions

### Enable Debug Visualization

Add to your baked prefab:

```csharp
// In editor, add AnimecsDebuggerAuthoring component
// Enable "Show States"
```

In Play mode, you'll see:
- Current state name
- Target state (if transitioning)
- Blend progress
- Animation time

### Check Parameter Values

```csharp
float currentSpeed = AnimecsController.GetFloat(entityManager, entity, speedHash);
Debug.Log($"Speed parameter: {currentSpeed}");
```

### Verify State Index

```csharp
var stateData = entityManager.GetComponentData<AnimecsStateData>(entity);
var stateMachineBlob = entityManager.GetComponentData<AnimecsStateMachineBlob>(entity);
ref var stateMachine = ref stateMachineBlob.StateMachineBlob.Value;

var currentState = AnimecsController.GetStateName(ref stateMachine, stateData.CurrentStateIndex);
Debug.Log($"Current: {currentState}, Transitioning: {AnimecsController.IsTransitioning(stateData)}");
```

## Common Issues

**Transition fires too early** - Increase exit time or add duration.

**Transition never fires** - Check all conditions are met. Verify parameter hash matches Animator Controller.

**Events missed** - Clear event buffer every frame. Events only fire once per crossing.

**Trigger doesn't work** - Triggers are consumed after transition. Set again if needed.

---

**Next:** [Blend Trees →](blend-trees.md) - Locomotion and smooth animation blending