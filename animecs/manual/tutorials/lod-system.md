# LOD System

Animecs uses distance-based LOD to reduce animation update frequency. Characters far from the camera update less often, saving CPU cycles without visual impact.

## How LOD Works

Every frame, `AnimecsLODSystem` calculates distance from each character to the camera and assigns an LOD level. This determines update frequency.

### Default Configuration

| LOD Level | Distance | Update Frequency | Use Case |
|-----------|----------|------------------|----------|
| High | 0-15m | Every frame | Player, nearby NPCs |
| Medium | 15-40m | Every 2 frames | Mid-distance crowds |
| Low | >40m | Every 4 frames | Background characters |
| Off | Frustum culled | No updates | Outside camera view |

### Temporal Distribution

Updates are distributed across frames using entity index. With 100 entities at Medium LOD:
- Frame 0: Entities 0, 2, 4, 6... update
- Frame 1: Entities 1, 3, 5, 7... update

This prevents frame spikes when many entities share the same LOD level.

## Camera Setup

LOD requires a camera reference.
Set it using `AnimecsController.SetLODCamera()` if you want to use a different camera, 
by default camera reference is set to `Camera.main`.

Without a camera (if null), LOD system uses default behavior (all entities High LOD).

## Customizing LOD Config

Create a system to modify the global LOD configuration:

```csharp
[UpdateInGroup(typeof(InitializationSystemGroup))]
public partial struct CustomLODConfigSystem : ISystem
{
    public void OnCreate(ref SystemState state)
    {
        state.RequireForUpdate<AnimecsLODConfig>();
    }

    public void OnUpdate(ref SystemState state)
    {
        var config = SystemAPI.GetSingleton<AnimecsLODConfig>();

        // Wider distance ranges for open-world game
        config.SetDistanceThresholds(
            high: 20f,     // Was 15m
            medium: 60f,   // Was 40m
        );

        // Aggressive performance mode
        config.SetUpdateFrequencies(
            high: 1,       // Every frame
            medium: 4,     // Every 4 frames (was 2)
            low: 16        // Every 16 frames (was 4)
        );

        SystemAPI.SetSingleton(config);

        // Run once
        state.Enabled = false;
    }
}
```

## Frustum Culling

Entities outside camera view are set to LOD Off automatically:
```csharp
var config = AnimecsController.GetLODConfig(entityManager);

// Enable frustum culling (enabled by default)
config.EnableFrustumCulling = true;

// Adjust radius threshold (default 2m)
config.FrustumCullingRadiusThreshold = 3f;

AnimecsController.SetLODConfig(entityManager, config);
```

## Manual LOD Override

Disable automatic LOD for specific entities:

```csharp
var entityManager = state.EntityManager;
var lod = entityManager.GetComponentData<AnimecsLOD>(entity);

// Disable auto LOD
lod.EnableAutoLOD = false;

// Force High LOD
AnimecsController.SetLODLevel(entityManager, entity, AnimecsLODLevel.High);
```

Re-enable auto LOD:

```csharp
var lod = entityManager.GetComponentData<AnimecsLOD>(entity);
lod.EnableAutoLOD = true;
entityManager.SetComponentData(entity, lod);
```

## Profiling LOD Impact

### Using Unity Profiler

1. Open Profiler (Window → Analysis → Profiler)
2. Find `AnimecsSystemGroup` in hierarchy
3. Check individual system costs:
   - `AnimecsLODSystem` - Distance calculation
   - `AnimecsStateProgressionSystem` - Time updates
   - `AnimecsSkinDeformationSystem` - Bone sampling

### Measuring Update Frequency

Count how many entities update per frame:

```csharp
[BurstCompile]
public partial struct LODStatsSystem : ISystem
{
    public void OnUpdate(ref SystemState state)
    {
        int totalEntities = 0;
        int highCount = 0;
        int mediumCount = 0;
        int lowCount = 0;
        int offCount = 0;

        foreach (var lod in SystemAPI.Query<RefRO<AnimecsLOD>>())
        {
            totalEntities++;
            switch (lod.ValueRO.Level)
            {
                case AnimecsLODLevel.High: highCount++; break;
                case AnimecsLODLevel.Medium: mediumCount++; break;
                case AnimecsLODLevel.Low: lowCount++; break;
                case AnimecsLODLevel.Off: offCount++; break;
            }
        }

        Debug.Log($"LOD Stats - High: {highCount}, Medium: {mediumCount}, " +
                  $"Low: {lowCount}, Off: {offCount}, Total: {totalEntities}");
    }
}
```

## LOD Behavior Details

### State Transitions

Transitions **respect LOD update frequency**:

```csharp
// Character at Medium LOD (updates every 2 frames)
AnimecsController.SetTrigger(entityManager, entity, attackHash);

// Transition evaluates every 2 frames
// Blend progress advances 2 frames worth of deltaTime
```

This maintains animation speed while reducing update frequency.

### Blend Trees

Blend tree weights update based on LOD:

```csharp
// High LOD: Parameters → Weights recalculated every frame
// Medium LOD: Weights recalculated every 2 frames
// Low LOD: Weights recalculated every 4 frames
```

Parameters can be set every frame, but weight evaluation is cached.

### Animation Events

Events **fire regardless of LOD**, but detection frequency matches update rate:

```csharp
// Low LOD entity (updates every 4 frames)
// Event at frame 45 might be detected at frame 43 or 47
```

For critical events (weapon hit, footstep), use High LOD or manual override.
Default settings work well for most games.

## Visualizing LOD Levels

Add debug visualization to see LOD distribution:

```csharp
public partial class LODDebugSystem : SystemBase
{
    protected override void OnUpdate()
    {
        Entities
            .WithAll<AnimecsTag>()
            .ForEach((in AnimecsLOD lod, in LocalToWorld transform) =>
            {
                Color color = lod.Level switch
                {
                    AnimecsLODLevel.High => Color.green,
                    AnimecsLODLevel.Medium => Color.yellow,
                    AnimecsLODLevel.Low => Color.red,
                    _ => Color.gray
                };

                Debug.DrawLine(
                    transform.Position,
                    transform.Position + new float3(0, 2, 0),
                    color
                );
            }).WithoutBurst().Run();
    }
}
```

Or use `AnimecsDebuggerAuthoring` component (shows LOD level in overlay).

## Best Practices

**Set realistic thresholds** - Test in-game camera distances. What looks acceptable at 40m in your game?

**Profile before optimizing** - Default settings work for most cases. Only change if profiler shows animation cost issues.

**Use manual override sparingly** - Let automatic LOD handle most entities. Override only for special cases.

**Consider gameplay camera** - Third-person camera keeps player close (High LOD). Top-down strategy might need different thresholds.

## Note
**Animecs not playing in SceneView?** - Those entities are frustum culled, and has LOD level set to Off.

---