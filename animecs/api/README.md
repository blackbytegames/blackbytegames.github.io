# Scripting API Overview

Runtime control API for Animecs animation system in DOTS.

## Core Modules

| Module | Purpose |
|--------|---------|
| [AnimecsController](controller.md) | Play states, set parameters, query animation data |
| [Components](components.md) | ECS component definitions |
| [Systems](systems.md) | ECS system execution |
| [Data Structures](data-structures.md) | Blob assets and runtime data |
| [Bakers](baker.md) | Authoring components for baking |

## Quick Reference

### Playing Animations

```csharp
// By name
AnimecsController.Play(ecb, index, entity, "Run", transitionDuration: 0.3f);

// By index
AnimecsController.Play(ecb, index, entity, stateIndex: 2, transitionDuration: 0.3f);
```

### Setting Parameters

```csharp
int speedHash = Animator.StringToHash("Speed");
AnimecsController.SetFloat(entityManager, entity, speedHash, 4.5f);
AnimecsController.SetTrigger(entityManager, entity, jumpHash);
```

### Reading State

```csharp
var stateData = entityManager.GetComponentData<AnimecsStateData>(entity);
int currentState = AnimecsController.GetCurrentStateIndex(stateData);
bool transitioning = AnimecsController.IsTransitioning(stateData);
```

### LOD Control

```csharp
AnimecsController.SetLODLevel(entityManager, entity, AnimecsLODLevel.High);
var config = AnimecsController.GetLODConfig(entityManager);
```

## Component Access Patterns

### Query State Machine Entities

```csharp
foreach (var (stateData, entity) in 
    SystemAPI.Query<RefRO<AnimecsStateData>>()
             .WithAll<AnimecsTag>()
             .WithEntityAccess())
{
    // Process animation state
}
```

### Access Parameters

```csharp
var floatBuffer = entityManager.GetBuffer<AnimecsFloatParameter>(entity);
var intBuffer = entityManager.GetBuffer<AnimecsIntParameter>(entity);
var boolBuffer = entityManager.GetBuffer<AnimecsBoolParameter>(entity);
```

### Process Events

```csharp
foreach (var (eventBuffer, entity) in 
    SystemAPI.Query<DynamicBuffer<AnimecsTriggeredEvent>>()
             .WithEntityAccess())
{
    foreach (var evt in eventBuffer)
    {
        // Handle event
    }
    eventBuffer.Clear();
}
```

## System Integration

Custom systems execute in `AnimecsSystemGroup`:

```csharp
[UpdateInGroup(typeof(AnimecsSystemGroup))]
[UpdateAfter(typeof(AnimecsStateProgressionSystem))]
public partial struct CustomAnimationSystem : ISystem
{
    public void OnUpdate(ref SystemState state)
    {
        // Custom animation logic
    }
}
```

## Burst Compatibility

All Animecs systems and `AnimecsController` methods are Burst-compatible. Use in jobs:

```csharp
[BurstCompile]
partial struct AnimationControlJob : IJobEntity
{
    public EntityCommandBuffer.ParallelWriter ECB;

    void Execute([EntityIndexInQuery] int index, Entity entity, in PlayerInput input)
    {
        if (input.Attack)
        {
            AnimecsController.Play(ECB, index, entity, "Attack", 0.1f);
        }
    }
}
```

## Performance Considerations

**Parameter Updates:** Setting parameters every frame is expected. Transition evaluation is Burst-compiled.

**State Queries:** Reading `AnimecsStateData` has zero overhead. It's a direct component read.

**LOD Impact:** Systems respect LOD update frequency automatically. Manual overrides bypass this.

**Event Processing:** Clear event buffers after processing to prevent duplicate handling.

---

**Next Steps:**

- [AnimecsController →](controller.md) - Full API reference
- [Components →](components.md) - Component details
- [Data Structures →](data-structures.md) - Blob asset layout
