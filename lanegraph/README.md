# Lane Graph Documentation

**Lane Graph** is a lightweight lane-based navigation system for Unity. Build complex road networks with paths, intersections, and transitions using visual editor tools, then query them at runtime with high-performance spatial lookups.

## Quick Links

- **New User?** Start with the [Quick Start Guide](manual/getting-started.md)
- **Understanding the System?** Read [Core Concepts](manual/architecture.md)
- **API Reference?** See [LaneGraphManager](api/manager.md)
- **Examples?** Check out the [Tutorials](manual/tutorials/basic-setup.md)

## Key Features

**High Performance**
- BVH-accelerated queries: O(log n) spatial lookups
- Tested with 50,000+ lanes per scene
- Zero garbage allocation during queries

**Visual Editor Tools**
- Real-time lane visualization with gizmos
- Automatic snapping between components
- Scene view manipulation with handles

**Flexible Design**
- Three component types (Path, Intersection, Transition)
- Reusable lane profiles
- Customizable tags and properties

**Runtime Control**
- Dynamic lane state changes
- Traffic signal management
- Event-driven state updates

## System Requirements

- **Unity:** 2021.3 or later
- **Dependencies:** None
- **Platforms:** All (PC, Console, Mobile, WebGL)
- **Rendering:** All pipelines (Built-in, URP, HDRP)

## Use Cases

LaneGraph excels at structured navigation where entities follow defined routes:

- **Racing Games** - Tracks with multiple racing lines and pit lanes
- **Traffic Simulation** - Realistic road networks with proper traffic flow
- **Strategy Games** - Unit movement corridors and pathfinding
- **Open World** - Vehicle AI that follows road networks

Unlike free-form navigation (NavMesh) or simple waypoints, LaneGraph provides:
- Explicit lane connectivity for pathfinding
- Direction constraints (forward/backward)
- Runtime state control (traffic lights, closures)
- Spatial efficiency via hierarchical acceleration

## Documentation Structure

### Getting Started
New to LaneGraph? These guides introduce core concepts:
- [What is LaneGraph?](manual/about.md) - Overview and use cases
- [Quick Start](manual/getting-started.md) - Build your first network
- [Core Concepts](manual/architecture.md) - System architecture

### User Guide
Detailed information on system features:
- [Lane Profiles](manual/lane-profiles.md) - Configure lane properties
- [Component Types](manual/component-types.md) - Path, Intersection, Transition
- [Editor Tools](manual/editor-tools.md) - Visual editing workflow
- [Best Practices](manual/best-practices.md) - Optimization and patterns

### Tutorials
Step-by-step guides for common tasks:
- [Basic Setup](manual/tutorials/basic-setup.md) - Create a simple path
- [Creating Paths](manual/tutorials/creating-paths.md) - Advanced path techniques
- [Building Intersections](manual/tutorials/building-intersections.md) - Multi-way junctions
- [Lane Transitions](manual/tutorials/lane-transitions.md) - Merges and splits
- [Traffic Control](manual/tutorials/traffic-control.md) - Signal management
- [Runtime Queries](manual/tutorials/runtime-queries.md) - API integration

### Scripting Reference
Complete API documentation:
- [LaneGraphManager](api/manager.md) - Core runtime API
- [Spatial Queries](api/spatial-queries.md) - Finding lanes
- [Lane Types](api/lane-types.md) - Data structures
- [Components](api/components.md) - Component interfaces

## Quick Example

Create a simple lane network and query it:

```csharp
using UnityEngine;
using LaneGraph;

public class VehicleNav : MonoBehaviour
{
    void Start()
    {
        // Initialize the system
        LaneGraphManager.Initialize();
        
        // Find closest lane
        int laneIndex = LaneGraphManager.FindClosestLaneIndex(
            transform.position,
            maxDistance: 50f,
            considerDirection: true,
            direction: transform.forward,
            tags: LaneTags.Vehicle
        );
        
        if (laneIndex >= 0)
        {
            // Get lane information
            var lane = LaneGraphManager.GetLane(laneIndex);
            if (lane.HasValue)
            {
                var points = lane.Value.LanePoints;
                var width = lane.Value.LaneConfiguration.Width;
                var speed = lane.Value.LaneConfiguration.Speed;
                
                // Use lane data for navigation
                NavigateAlongLane(points, speed);
            }
        }
    }
}
```

## Performance Characteristics

### Spatial Query Complexity
```
Traditional Waypoints:  O(n) linear search
Simple Splines:         O(n) per query
LaneGraph BVH:          O(log n + k·m) where k << n
```

For a scene with 1,000 components and 10,000 lanes:
- Linear search: ~10,000 checks per query
- LaneGraph: ~30-50 checks per query

This 200-300x improvement enables real-time queries for complex scenes.

### Memory Usage
Typical lane (20 points, 3 connections): ~300 bytes

Scene examples:
- Small (100 lanes): ~30 KB
- Medium (1,000 lanes): ~300 KB
- Large (10,000 lanes): ~3 MB

## Support

**Email:** sridogames@gmail.com  
**GitHub:** [Blackbyte Games](https://github.com/blackbytegames)  
**Asset Store:** [View on Unity Asset Store](https://assetstore.unity.com)

## Version

Current version: **1.5.0**

See [Changelog](changelog.md) for version history and updates.

---

Ready to build your lane network? Start with the [Quick Start Guide](manual/getting-started.md).
