# LaneGraph API Reference

Complete API documentation for LaneGraph runtime and editor systems.

## Core Runtime API

### [LaneGraphManager](manager.md)
The main interface for all runtime operations. Handles initialization, queries, and lane state management.

**Key Methods:**
- `Initialize()` - Load lane graph for current scene
- `FindClosestLaneIndex()` - Spatial queries
- `GetLane()` - Access lane data
- `SetLaneState()` - Control lane availability

### [Lane Types](lane-types.md)
Core data structures representing lanes and their properties.

**Structures:**
- `Lane` - Complete lane definition
- `LaneConfiguration` - Lane properties
- `LaneLinker` - Connection between lanes
- `LaneComponentData` - Component metadata

### [Spatial Queries](spatial-queries.md)
High-performance BVH-based spatial query system.

**Operations:**
- Closest lane queries
- Radius searches
- Direction filtering
- Tag-based filtering

### [Lane State Control](lane-state.md)
Dynamic lane state management for traffic control.

**Features:**
- Runtime state changes
- Signal group management
- Event callbacks
- Traffic light simulation

## Component API

### [Components](components.md)
Editor components for building lane networks.

**Types:**
- `PathComponent` - Basic paths
- `IntersectionComponent` - Multi-way junctions
- `LaneTransitionComponent` - Merges and splits

## Enumerations

### LaneTags
```csharp
[Flags]
public enum LaneTags
{
    None = 0,
    Vehicle = 1 << 0,
    Pedestrian = 1 << 1,
    Bicycle = 1 << 2,
    Emergency = 1 << 3
}
```

### LaneState
```csharp
public enum LaneState
{
    Open = 0,
    Closed = 1,
    AboutToClose = 2,
    Yield = 3
}
```

### LaneDirection
```csharp
public enum LaneDirection
{
    None = 0,
    Forward = 1,
    Backward = 2
}
```

### LaneLinkType
```csharp
[Flags]
public enum LaneLinkType
{
    None = 0,
    Forward = 1 << 0,
    Merge = 1 << 1,
    Split = 1 << 2,
    Intersection = 1 << 3
}
```

## Quick Reference

### Common Operations

**Initialize system:**
```csharp
LaneGraphManager.Initialize();
```

**Find closest lane:**
```csharp
int lane = LaneGraphManager.FindClosestLaneIndex(position);
```

**Get lane info:**
```csharp
Lane? lane = LaneGraphManager.GetLane(laneIndex);
```

**Change lane state:**
```csharp
LaneGraphManager.SetLaneState(laneIndex, LaneState.Closed);
```

**Find lanes in area:**
```csharp
int[] lanes = LaneGraphManager.FindLanesInRadius(position, radius);
```

## Namespace

All LaneGraph APIs are in the `LaneGraph` namespace:

```csharp
using LaneGraph;
using Unity.Mathematics; // For float3
```

## Thread Safety

LaneGraph is **not thread-safe**. All API calls must be made from the main Unity thread.

## Performance Notes

- Queries use BVH acceleration structure
- O(log n) for spatial queries
- O(1) for lane property access
- O(1) for state changes

## See Also

- [Getting Started Guide](../manual/getting-started.md)
- [Runtime Queries Tutorial](../manual/tutorials/runtime-queries.md)
- [Architecture Overview](../manual/architecture.md)
