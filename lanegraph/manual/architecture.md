# Core Concepts

Understanding Lane Graph's architecture helps you build efficient lane networks and integrate them with your game systems.

## System Architecture

```
Editor Time                  Build Time                    Runtime
┌──────────────┐            ┌──────────────┐             ┌──────────────┐
│ Components   │            │ Graph        │             │ Manager      │
│ - Path       │───build───>│ Builder      │───creates──>│ - Initialize │
│ - Intersection            │(ScriptableObject)          │ - Query      │
│ - Transition │            │              │             │ - Control    │
└──────────────┘            └──────────────┘             └──────────────┘
       │                                                         │
       ↓                                                         ↓
┌──────────────┐                                         ┌──────────────┐
│ Lane Profiles│                                         │ BVH System   │
│ (Project     │                                         │ (Spatial     │
│  Settings)   │                                         │  Queries)    │
└──────────────┘                                         └──────────────┘
```

## Data Flow

### 1. Editor Time
Create and configure components using visual tools. Each component references a Lane Profile and generates preview lanes in real-time.

### 2. Build Time
The Graph Builder scans all lanes in the scene, collects component data, and generates optimized runtime structures:
- Flattened lane array
- Component metadata
- Scene index mapping
- Spatial acceleration structure

### 3. Runtime
The Manager initializes the current scene's data and provides query APIs. The BVH system accelerates spatial searches.

## Component Types

### PathComponent
Represents a path with parallel lanes. Supports three shape types:

**Linear** - Straight segments between nodes
```
Node ●──────●──────● Node
Lane ────────────────
Lane ────────────────
```

**AutoBezier** - Automatically smoothed curves
```
Node ●╭───╮●╭───╮● Node
Lane ─╯   ╰─╯   ╰─
Lane ─╯   ╰─╯   ╰─
```

**Bezier** - Manual tangent control for precise curves
```
Node ●╮    ╭●╮    ╭● Node
     ╱│╲  ╱ │╲  ╱
Lane╯ │ ╲╯  │ ╲╯
Lane╯ │ ╲╯  │ ╲╯
```

### IntersectionComponent
Multi-way junction with per-node profiles and configurable lane connections. Each node can have different lane counts and configurations.

**Key features:**
- 3+ node points forming the junction
- Per-node lane profiles
- Explicit lane-to-lane connections
- Automatic curve generation between nodes

### LaneTransitionComponent
Handles lane count changes through merges and splits.

**Merge** - Multiple lanes combine into fewer
```
Lane ─╮
Lane ─┤──> Merged Lane
Lane ─╯
```

**Split** - Lane divides into multiple
```
              ╭─ Lane
Merged Lane ──┼─ Lane
              ╰─ Lane
```

## Lane Profiles

Lane Profiles are templates defining lane structure:

```csharp
LaneConfiguration
{
    float Width;             // Physical width
    LaneDirection Direction; // Forward/Backward
    LaneTags Tags;           // Custom categories
    float Speed;             // Speed limit
    LaneState LaneState;     // Default state (traffic signal control)
}
```

**Tag System**
Tags are flags enabling multiple categories per lane. Configure tag names in Project Settings > Lane Graph > Lane Tags section.

**Example tags:**
- Default (enabled by default)
- Vehicle
- Pedestrian
- Emergency
- Custom1, Custom2, etc.

Tags enable filtering queries: "Find only vehicle lanes" or "Query pedestrian paths only."

## Spatial Query System

### Two-Level BVH

LaneGraph uses hierarchical bounding volumes for O(log n) queries:

**Level 1: Component BVH**
Groups components spatially using axis-aligned bounding boxes. Quickly rejects distant components.

**Level 2: Lane Queries**
Within candidate components, performs point-to-lane distance calculations only for relevant lanes.

### Query Performance

```
FindClosestLane(position):
  1. BVH traversal → O(log n) candidate components
  2. AABB distance checks → O(k) where k = candidates
  3. Lane distance calculations → O(m) where m = lanes per component
  
Total: O(log n + k·m)
```

For typical scenes (n=1000 components, k=3-5 candidates, m=10 lanes):
- Traditional linear search: ~10,000 lane checks
- LaneGraph BVH: ~30-50 lane checks

This 200-300x reduction enables real-time queries for large scenes.

## Lane Connections

Lanes connect through **LaneLinkers**:

```csharp
struct LaneLinker
{
    int LaneIndex;        // Target lane
    LaneLinkType LinkType; // Connection type
}
```

**Link Types:**
- **Forward** - Natural continuation
- **Merge** - Transition merge
- **Split** - Transition split  
- **Intersection** - Junction connection

The system maintains a directed graph of connections, enabling pathfinding algorithms.

## Lane States

Lanes have operational states changed at runtime:

- **Open** - Normal operation
- **Closed** - Blocked/unavailable (red light)
- **AboutToClose** - Warning (yellow light)
- **Yield** - Proceed with caution
- and more.

The LaneStateController manages signal groups with automatic cycling:

```
Signal Group → [Lane1, Lane2, Lane3]
├─ Green Duration: 30s
├─ Yellow Duration: 3s
└─ Current State: Open

Timeline:
[Open 30s] → [AboutToClose 3s] → [Closed] → [Open 30s] → ...
```

## Extension Points

### Custom Pathfinding
Implement A* from the plugin or your preferred algorithm using lane connections:

```csharp
public static int[] FindPath(int startLane, int goalLane)
{
    // Use LaneGraphManager.GetLaneLinkers() to get connections
    // Implement your pathfinding logic
    // Return lane index array
}
```

### Custom State Logic
Extend LaneState enum and implement custom controllers:

```csharp
public enum LaneState
{
    // Built-in states
    Open = 0,
    Closed = 1,
    
    // Your custom states
    UnderConstruction = 10,
    EmergencyVehicleOnly = 11,
}
```

### Event Integration
React to state changes for AI, visuals, or gameplay:

```csharp
LaneGraphManager.OnLaneStateChanged += (laneIndex, newState) =>
{
    if (newState == LaneState.Closed)
    {
        // Reroute traffic
        // Update visuals
        // Notify AI systems
    }
};
```

---

This architecture enables LaneGraph to handle complex networks efficiently while remaining simple to integrate and extend.
