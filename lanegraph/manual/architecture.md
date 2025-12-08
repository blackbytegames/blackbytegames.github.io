# LaneGraph Architecture

This document explains the technical architecture of LaneGraph, helping you understand how the system works under the hood.

## System Overview

LaneGraph is built on three main pillars:

1. **Editor Tools**: For designing lane networks visually
2. **Data Management**: For storing and organizing lane information
3. **Runtime Query System**: For fast spatial queries and pathfinding

## Component Architecture

### Hierarchy Structure

```
ILaneComponent (Interface)
    │
    ├── PathComponent
    │   - Simple paths with multiple lanes
    │   - Shape types: Linear, AutoBezier, Bezier
    │   - Node-based editing
    │
    ├── IntersectionComponent
    │   - Multi-node junctions
    │   - Per-node lane profiles
    │   - Configurable lane connections
    │
    └── LaneTransitionComponent
        - Merge/Split transitions
        - Gradual lane count changes
        - Smooth blending
```

### Core Data Structures

```csharp
// Lane - Represents a single lane
struct Lane
{
    int LaneIndex;                    // Global index
    LaneConfiguration Configuration;   // Width, direction, tags, etc.
    float3[] LanePoints;              // Path geometry
    LaneState CurrentLaneState;       // Runtime state
    LaneLinker[] LaneLinkers;         // Connections
    bool IsIntersectionLane;          // Special handling flag
}

// LaneConfiguration - Lane properties
struct LaneConfiguration
{
    float Width;                      // Lane width
    LaneDirection Direction;          // Forward/Backward
    LaneTags Tags;                    // Custom categories
    float Speed;                      // Speed limit
    LaneState DefaultState;           // Initial state
}

// LaneComponentData - Component metadata
struct LaneComponentData
{
    int ComponentId;                  // Unique identifier
    int[] LanesIndex;                 // Indices of lanes in this component
    float3[] BoundaryPoints;          // AABB boundary
}
```

## Data Flow

### Editor Time

1. **User creates/modifies components** in Scene view
2. **Components generate lanes** based on profiles
3. **Visual feedback** via gizmos and handles
4. **Validation** checks for errors

### Build Time

1. **LaneGraphBuilder** scans all scenes in build settings
2. **Collects all lane components** and their data
3. **Generates optimized data structures**:
   - Centralized lane array
   - Component metadata array
   - Scene index mapping
4. **Creates LaneGraphRegistry** asset
5. **Builds BVH spatial index**

### Runtime

1. **Initialize** loads registry for current scene
2. **BVH construction** for fast queries
3. **Query operations** use BVH traversal
4. **State changes** update lane array directly

## Spatial Query System

### BVH Structure

LaneGraph uses a two-level Bounding Volume Hierarchy:

```
Level 1: Component BVH
    - Groups components spatially
    - Broad-phase filtering
    - Quick rejection of distant areas

Level 2: Lane Queries
    - Within matching components
    - Fine-grained distance calculations
    - Direction and tag filtering
```

### Query Algorithm

```
FindClosestLane(position, maxDistance, direction, tags):
    1. Query component BVH for candidates
    2. For each candidate component:
        a. Check AABB distance
        b. If within range, check each lane
        c. Calculate point-to-lane distance
        d. Apply direction filter if needed
        e. Apply tag filter if needed
    3. Return closest matching lane
```

### Performance Characteristics

- **Component BVH Construction**: O(n log n)
- **Component Query**: O(log n)
- **Lane Distance Calculation**: O(m) where m = points per lane
- **Overall Query**: O(log n + k·m) where k = candidate components

## Lane State Management

### State Controller System

```
LaneStateController
    │
    ├── SignalGroup 1
    │   ├── Lane Indices
    │   ├── Timing Configuration
    │   └── Current State
    │
    ├── SignalGroup 2
    └── SignalGroup N
```

### State Transitions

```
Green (Open)
    ↓ (yellow duration)
Yellow (AboutToClose)
    ↓ (automatic)
Red (Closed)
    ↓ (green duration)
Back to Green
```

### Event System

```csharp
// Register for state change notifications
LaneGraphManager.OnLaneStateChanged += (laneIndex, newState) =>
{
    // React to state changes
    Debug.Log($"Lane {laneIndex} is now {newState}");
};
```

## Memory Management

### Storage Layout

```
LaneGraphManager (Static)
    - Lane[] Lanes
    - LaneComponentData[] ComponentData
    - int CurrentSceneIndex

LaneGraphRegistry (ScriptableObject)
    - SceneData[] PerSceneData
    - Global configuration

BVHSystem (Static)
    - BVHNode tree structure
    - ComponentBVHData[] array
    - float3[] lane directions cache
```

### Memory Optimization Strategies

1. **Struct-based design** - No heap allocations for core types
2. **Array pooling** - Reuse temporary arrays in queries
3. **Stackalloc** - Use stack memory for small temporary buffers
4. **Lazy BVH build** - Build only when first query is made
5. **Per-scene isolation** - Load only relevant data

## Editor Architecture

### Inspector Editors

```
LaneComponentEditorBase<T>
    │
    ├── PathComponentEditor
    ├── IntersectionComponentEditor
    └── TransitionComponentEditor
```

### Scene View Tools

```
LaneComponentTransformer
    - Detects transform changes
    - Triggers lane regeneration

LaneComponentSelectionMonitor
    - Tracks selected components
    - Handles multi-selection

LaneSnappingHandler
    - Detects nearby endpoints
    - Provides snap hints

LaneGizmosDrawer
    - Draws lane visualizations
    - Distance-based LOD
    - Color-coded by tag
```

## Extension Points

### Custom Lane Tags

```csharp
[Flags]
public enum LaneTags
{
    None = 0,
    Vehicle = 1 << 0,
    Pedestrian = 1 << 1,
    Bicycle = 1 << 2,
    Emergency = 1 << 3,
    // Add your custom tags here
    Custom1 = 1 << 8,
    Custom2 = 1 << 9,
}
```

### Custom Lane States

```csharp
public enum LaneState
{
    Open = 0,
    Closed = 1,
    AboutToClose = 2,
    Yield = 3,
    // Add your custom states here
    Maintenance = 10,
    Reserved = 11,
}
```

### Custom Pathfinding

```csharp
// Implement your own pathfinding logic
public static List<int> FindPath(int startLane, int endLane)
{
    // Use LaneGraphManager API to query lanes
    // Implement A* or your preferred algorithm
    // Return list of lane indices
}
```

## Performance Considerations

### Best Practices

1. **Batch queries** - Group multiple queries together when possible
2. **Cache results** - Store frequently used lane indices
3. **Use appropriate maxDistance** - Smaller distances = faster queries
4. **Leverage tag filtering** - Reduces candidate count
5. **Initialize early** - Call Initialize() before first query
6. **Build once per session** - Don't rebuild in play mode

### Profiling Tips

- Monitor `LaneGraphManager.Initialize()` time
- Profile `FindClosestLaneIndex()` calls
- Check BVH construction time with large networks
- Measure memory usage of lane arrays

### Scalability Limits

- Tested with **50,000+ lanes** per scene
- BVH can handle **10,000+ components** efficiently
- Query performance remains consistent at scale
- Memory scales linearly with lane count

## Thread Safety

LaneGraph is **not thread-safe** by default. Follow these guidelines:

- **Main thread only**: All LaneGraphManager calls
- **Read-only queries**: Safe after initialization
- **State changes**: Must be on main thread
- **Batch processing**: Can distribute work across frames

For multi-threaded scenarios:
```csharp
// Store lane data in local variables
var lanePoints = LaneGraphManager.GetLanePoints(laneIndex);

// Process on worker thread
ThreadPool.QueueUserWorkItem(_ =>
{
    // Use local copy safely
    ProcessLanePoints(lanePoints);
});
```

## Debugging Tools

### Debug Logging

```csharp
// Enable verbose logging
LaneGraphDebug.SetVerbosity(LogVerbosity.Verbose);

// Log specific operations
LaneGraphDebug.ConditionalLog("Custom message", LogVerbosity.Info);
```

### Visual Debugging

```csharp
// Enable visual debug features
LaneGraphDebugSettings.ShowLaneBounds = true;
LaneGraphDebugSettings.ShowLaneConnections = true;
LaneGraphDebugSettings.ShowComponentIds = true;
```

### Gizmo Settings

Adjust in `LaneGraphEditorSettings`:
- MaxGizmoDrawDistance: Cull distant lanes
- HandleSizeMagnitudeFactor: Adjust handle sizes
- Lane color coding: Customize tag colors

---

This architecture enables LaneGraph to provide high-performance lane-based navigation while maintaining flexibility and ease of use. Understanding these concepts will help you make the most of the system.
