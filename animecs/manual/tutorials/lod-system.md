# LOD System

Animecs uses distance-based LOD to reduce animation update frequency. Characters far from the camera update less often, saving CPU cycles without visual impact.

## How LOD Works

Every frame, `AnimecsLODSystem` calculates distance from each character to the camera and assigns an LOD level. This determines update frequency.

### Default Configuration

| LOD Level | Distance | Update Frequency | Use Case |
|-----------|----------|------------------|----------|
| High | 0-15m | Every frame | Player, nearby NPCs |
| Medium | 15-40m | Every 2 frames | Mid-distance crowds |
| Low | 40-100m | Every 8 frames | Background characters |
| Off | >100m | No updates | Culled entities |

### Temporal Distribution

Updates are distributed across frames using entity index. With 100 entities at Medium LOD:
- Frame 0: Entities 0, 2, 4, 6... update
- Frame 1: Entities 1, 3, 5, 7... update

This prevents frame spikes when many entities share the same LOD level.

## Camera Setup

LOD requires a camera with `AnimecsCameraTag`. Add to your main camera entity:

```csharp
// In authoring component
public class MainCameraAuthoring : MonoBehaviour
{
    class Baker : Baker<MainCameraAuthoring>
    {
        public override void Bake(MainCameraAuthoring authoring)
        {
            var entity = GetEntity(TransformUsageFlags.Dynamic);
            AddComponent<AnimecsCameraTag>(entity);
        }
    }
}
```

Without `AnimecsCameraTag`, LOD system won't initialize and all entities run at High LOD.

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
            low: 150f      // Was 100m
        );

        // Aggressive performance mode
        config.SetUpdateFrequencies(
            high: 1,       // Every frame
            medium: 4,     // Every 4 frames (was 2)
            low: 16        // Every 16 frames (was 8)
        );

        SystemAPI.SetSingleton(config);

        // Run once
        state.Enabled = false;
    }
}
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

### Use Cases for Manual LOD

**Boss entities** - Always High LOD regardless of distance:
```csharp
if (entityManager.HasComponent<BossTag>(entity))
{
    AnimecsController.SetLODLevel(entityManager, entity, AnimecsLODLevel.High);
}
```

**Cutscene characters** - Force High LOD during cinematics:
```csharp
if (inCutscene)
{
    foreach (var entity in cutsceneCharacters)
    {
        AnimecsController.SetLODLevel(entityManager, entity, AnimecsLODLevel.High);
    }
}
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

### Expected Performance

With 1000 entities:
- High LOD (50 entities): 50 updates/frame
- Medium LOD (200 entities): 100 updates/frame (200÷2)
- Low LOD (500 entities): 62.5 updates/frame (500÷8)
- Off LOD (250 entities): 0 updates/frame

**Total: ~212 animation updates/frame instead of 1000**

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
// Low LOD: Weights recalculated every 8 frames
```

Parameters can be set every frame, but weight evaluation is cached.

### Animation Events

Events **fire regardless of LOD**, but detection frequency matches update rate:

```csharp
// Low LOD entity (updates every 8 frames)
// Event at frame 45 might be detected at frame 40 or 48
```

For critical events (weapon hit, footstep), use High LOD or manual override.

## Balancing LOD Settings

### Conservative (Quality)

```csharp
config.SetDistanceThresholds(high: 25f, medium: 60f, low: 120f);
config.SetUpdateFrequencies(high: 1, medium: 2, low: 4);
```

More entities at high quality, smoother animations overall.

### Aggressive (Performance)

```csharp
config.SetDistanceThresholds(high: 10f, medium: 30f, low: 80f);
config.SetUpdateFrequencies(high: 1, medium: 4, low: 16);
```

Fewer high-quality entities, maximum performance for crowds.

### Hybrid (Recommended)

```csharp
config.SetDistanceThresholds(high: 15f, medium: 40f, low: 100f);
config.SetUpdateFrequencies(high: 1, medium: 2, low: 8);
```

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

## Common Issues

**All entities at High LOD** - Camera missing `AnimecsCameraTag`. LOD system won't initialize.

**Jerky animations at distance** - Update frequency too low. Increase Medium/Low LOD update rates or adjust distance thresholds.

**Performance worse with LOD** - Too many entities in High LOD range. Reduce `highDistance` threshold.

**Events missed** - Critical events on Low LOD entities. Force High LOD for entities with important events.

---