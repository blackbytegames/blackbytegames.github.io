# About Lane Graph

Lane Graph is a lightweight, high-performance lane-based navigation system for Unity. It enables structured pathfinding along defined routes, making it ideal for racing games, traffic simulations, strategy games, and any application requiring organized movement paths.

## Why Lane-Based Navigation?

Traditional navigation systems like NavMesh, spline-points or waypoints work well for free-form movement, but fall short when entities must follow specific routes with defined rules. Lane Graph solves this by representing your network as lanes with explicit connections, directions, and properties.

**Key advantages:**
- **Predictable paths** – Objects/Entities follow well-defined routes
- **Direction control** – Forward/backward flow per lane
- **Traffic rules** – Lane states enable signals and restrictions
- **Spatial efficiency** – Bounding Volume Hierarchy (BVH) system acceleration provides O(log n) queries
- **Zero dependencies** – Standalone system with no external requirements

## Performance

Lane Graph uses a two-level Bounding Volume Hierarchy (BVH) for spatial queries:

1. **Component-level BVH** - Rapidly filters distant components
2. **Lane-level queries** - Fine-grained searches within relevant components

This architecture delivers:
- **Fast initialization** - O(n log n) construction
- **Efficient queries** - O(log n + m) where m is lanes in nearby components
- **Scalable** - Tested with 50,000+ lanes per scene
- **Constant state changes** - O(1) lane state updates

Unlike simple waypoint systems that require linear searches, or spline-based approaches that lack connectivity data, Lane Graph maintains both spatial efficiency and rich semantic information about lane relationships.

## Core Features

### Lane Profiles
Define reusable lane configurations including width, direction, speed limits, and custom tags. Create profiles for different road types (highways, city streets, pedestrian paths) and apply them across your network.

### Component Types
Build networks using three specialized components:
- **Path Component** - Straight or curved paths with multiple lanes
- **Intersection Component** - Multi-way junctions with configurable connections
- **Lane Transition Component** - Smooth merges and splits for lane count changes

### Runtime Control
Dynamically modify lane states for traffic lights, road closures, or gameplay events. The system provides event callbacks for state changes, enabling reactive AI and visual feedback.

### Editor Integration
Visual scene tools with gizmo rendering, automatic snapping, and real-time validation. Build and test your network directly in the Unity Editor with immediate visual feedback.

## Use Cases

**Traffic Simulation** - Build realistic road networks with proper intersections, lane merges, and traffic control.

**Racing Games** - Create tracks with multiple racing lines, pit lanes, and dynamic shortcuts.

**Strategy Games** - Define movement corridors with choke points and branching paths for unit AI.

**Open World** - Implement vehicle navigation along roads with proper lane discipline and traffic rules.

## System Requirements

- Unity 2019.1 or later
- No external dependencies
- All rendering pipelines (Built-in, URP, HDRP)
- All platforms (PC, Console, Mobile, WebGL)

## Next Steps

New to LaneGraph? Start with the [Quick Start](getting-started.md) guide to build your first lane network in minutes.

Ready to dive deeper? Explore [Core Concepts](architecture.md) to understand the system architecture.
