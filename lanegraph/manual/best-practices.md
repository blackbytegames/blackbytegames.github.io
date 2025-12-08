# Best Practices

Follow these guidelines for optimal results with LaneGraph.

## Quick Reference

✅ **DO:**
- Initialize once at scene start
- Cache frequently queried lane data
- Use appropriate maxDistance in queries
- Build graph after every change
- Name components descriptively
- Test incrementally

❌ **DON'T:**
- Query without initializing
- Build during play mode
- Use unnecessarily large max distances
- Query every frame if avoidable
- Create overly complex single components

## Performance

### Query Optimization
```csharp
// ✅ Cache lane data
private float3[] cachedPoints;
void Start() {
    cachedPoints = LaneGraphManager.GetLanePoints(laneIndex);
}

// ❌ Query every frame
void Update() {
    var points = LaneGraphManager.GetLanePoints(laneIndex);
}
```

### Appropriate Distances
```csharp
// ✅ Targeted search
FindClosestLaneIndex(pos, maxDistance: 20f);

// ❌ Wasteful
FindClosestLaneIndex(pos, maxDistance: 10000f);
```

## Design Patterns

### Initialization
```csharp
void Start()
{
    if (!LaneGraphManager.IsInitialized)
        LaneGraphManager.Initialize();
}
```

### Error Handling
```csharp
int lane = LaneGraphManager.FindClosestLaneIndex(pos);
if (lane >= 0)
{
    // Use lane safely
}
```

### Event Management
```csharp
void OnEnable()
{
    LaneGraphManager.OnLaneStateChanged += HandleStateChange;
}

void OnDisable()
{
    LaneGraphManager.OnLaneStateChanged -= HandleStateChange;
}
```

## Component Design

- Keep paths under 100 nodes
- Split long roads into segments
- Use consistent lane widths
- Test connections visually

## See Also

- [Architecture](architecture.md)
- [Runtime Queries Tutorial](tutorials/runtime-queries.md)
- [Limitations](limitations.md)
