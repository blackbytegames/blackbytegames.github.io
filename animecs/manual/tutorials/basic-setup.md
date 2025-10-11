# Basic Setup

This tutorial shows you how to control animations at runtime. You'll learn to play states, set parameters, and read current animation data.

## Prerequisites

- Baked Animecs prefab in a subscene
- Basic understanding of DOTS systems and EntityManager

## Playing States

Use `AnimecsController.Play()` to transition between states. There are two ways: by name or by index.

### By State Name

```csharp
[BurstCompile]
public partial struct PlayerInputSystem : ISystem
{
    public void OnUpdate(ref SystemState state)
    {
        var ecb = SystemAPI.GetSingleton<EndSimulationEntityCommandBufferSystem.Singleton>()
            .CreateCommandBuffer(state.WorldUnmanaged).AsParallelWriter();

        var job = new ProcessInputJob { ECB = ecb };
        job.ScheduleParallel();
    }
}

[BurstCompile]
partial struct ProcessInputJob : IJobEntity
{
    public EntityCommandBuffer.ParallelWriter ECB;

    void Execute([EntityIndexInQuery] int index, Entity entity, in PlayerInput input)
    {
        if (input.JumpPressed)
        {
            // Transition to Jump state over 0.1 seconds
            AnimecsController.Play(ref ECB, index, in entity, "Jump", 
                transitionDuration: 0.1f,
                curveType: AnimecsCurveType.EaseOut,
                overrideExitTime: true,
                resetTime: true);
        }
    }
}
```

### By State Index

If you know the state index, this is faster (no string lookup during baking):

```csharp
// State indices (set these based on your Animator Controller order)
const int IdleIndex = 0;
const int WalkIndex = 1;
const int RunIndex = 2;

void Execute([EntityIndexInQuery] int index, Entity entity, in MovementData movement)
{
    if (movement.Speed > 5f)
    {
        AnimecsController.Play(ref ECB, index, in entity, RunIndex, 
            transitionDuration: 0.3f);
    }
}
```

## Transition Parameters

`Play()` accepts several parameters:

**transitionDuration** - Blend time in seconds (0 = instant)  
**curveType** - Blend curve:
- `Linear` - Constant speed
- `Smooth` - Ease in and out (default)
- `EaseIn` - Slow start, fast end
- `EaseOut` - Fast start, slow end

**overrideExitTime** - Ignore exit time setting in transition  
**resetTime** - Start target state at time 0

## Setting Parameters

Parameters drive transitions. Set them based on gameplay logic.

### Float Parameters

**Main Thread:**
```csharp
var entityManager = state.EntityManager;
int speedHash = Animator.StringToHash("Speed");

AnimecsController.SetFloat(entityManager, entity, speedHash, 4.5f);
```
**Burst Job:**
```csharp
// In system OnCreate/OnUpdate
_floatLookup = state.GetBufferLookup<AnimecsFloatParameter>();
_parametersLookup = state.GetComponentLookup<AnimecsParameters>(true);

// In job Execute
ref var paramBlob = ref _parametersLookup[entity].ParameterBlob.Value;
int speedHash = Animator.StringToHash("Speed");
AnimecsController.SetFloat(ref _floatLookup, ref paramBlob, in entity, speedHash, 4.5f);
```

### Int Parameters
**Main Thread:**
```csharp
int weaponTypeHash = Animator.StringToHash("WeaponType");
AnimecsController.SetInt(entityManager, entity, weaponTypeHash, 2);
```
**Burst Job:**
```csharp
ref var paramBlob = ref _parametersLookup[entity].ParameterBlob.Value;
AnimecsController.SetInt(ref _intLookup, ref paramBlob, in entity, weaponTypeHash, 2);
```
### Bool Parameters
**Main Thread:**
```csharp
int groundedHash = Animator.StringToHash("Grounded");
AnimecsController.SetBool(entityManager, entity, groundedHash, true);
```
**Burst Job:**
```csharp
ref var paramBlob = ref _parametersLookup[entity].ParameterBlob.Value;
AnimecsController.SetBool(ref _boolLookup, ref paramBlob, in entity, groundedHash, true);
```
### Trigger Parameters

Triggers fire once, then reset automatically:

**Main Thread:**
```csharp
int attackHash = Animator.StringToHash("Attack");
AnimecsController.SetTrigger(entityManager, entity, attackHash);
```
**Burst Job:**
```csharp
ref var paramBlob = ref _parametersLookup[entity].ParameterBlob.Value;
AnimecsController.SetTrigger(ref _triggerLookup, ref paramBlob, in entity, attackHash);
```
## Reading Animation State

Query current animation data to sync gameplay with animation.

### Current State

```csharp
var stateData = entityManager.GetComponentData<AnimecsStateData>(entity);
int currentIndex = AnimecsController.GetCurrentStateIndex(stateData);
float currentTime = AnimecsController.GetCurrentStateTime(stateData);
```

### Transition Status

```csharp
if (AnimecsController.IsTransitioning(stateData))
{
    float progress = AnimecsController.GetTransitionProgress(stateData);
    // progress is 0.0 to 1.0
}
```

### State Name Lookup

If you need the name (for debugging):

```csharp
var stateMachineBlob = entityManager.GetComponentData<AnimecsStateMachineBlob>(entity);
ref var stateMachine = ref stateMachineBlob.StateMachineBlob.Value;

var stateName = AnimecsController.GetStateName(ref stateMachine, currentIndex);
Debug.Log($"Current state: {stateName}");
```

## Pausing Animation

Stop time progression without disabling the entity:

```csharp
var stateData = entityManager.GetComponentData<AnimecsStateData>(entity);
AnimecsController.SetStatePaused(ref stateData, paused: true);
entityManager.SetComponentData(entity, stateData);
```

Resume:

```csharp
AnimecsController.SetStatePaused(ref stateData, paused: false);
entityManager.SetComponentData(entity, stateData);
```

## Common Issues

**"Parameter not found"** - Your Animator Controller doesn't have this parameter. Check spelling and hash calculation.

**State won't change** - Verify transition conditions in your Animator Controller. Use debug visualization to see current state.

**Instant transitions when you want blending** - Check your `transitionDuration` value. Zero means instant.

---