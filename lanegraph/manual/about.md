# About LaneGraph

LaneGraph is a lightweight, high-performance lane-based navigation system designed specifically for Unity developers who need structured pathfinding solutions. Whether you're creating a racing game, traffic simulation, or any application requiring organized movement paths, LaneGraph provides the tools to build, visualize, and query complex lane networks.

## What Makes LaneGraph Different?

Unlike traditional nav mesh or grid-based pathfinding systems, LaneGraph focuses on **lane-based navigation** where entities follow defined paths with specific rules, directions, and behaviors. This makes it ideal for scenarios where movement should be constrained to specific routes.

### Core Philosophy

1. **Editor-First Design**: Build your lane networks visually in the Unity editor with intuitive tools
2. **Performance-Oriented**: Optimized spatial queries using hierarchical BVH structures
3. **Flexibility**: Three component types (Paths, Intersections, Transitions) cover virtually any use case
4. **Runtime Control**: Dynamic lane state changes for traffic lights, road closures, and more

## Key Concepts

### Lane Profiles

Lane Profiles define the structure and properties of lanes:
- **Width**: Physical width of each lane
- **Direction**: Forward or backward flow
- **Tags**: Custom categories (Vehicle, Pedestrian, Emergency, etc.)
- **Speed**: Default speed limit
- **State**: Operational status (Open, Closed, Yield, etc.)

### Component Types

LaneGraph provides three types of components that work together:

**Path Component**
- Basic building block for straight or curved lanes
- Supports Linear, AutoBezier, and Bezier shapes
- Can have multiple lanes side-by-side
- Ideal for roads, tracks, or corridors

**Intersection Component**
- Connects multiple paths together
- Each node can have its own lane profile
- Configurable lane-to-lane connections
- Perfect for crossroads, junctions, roundabouts

**Transition Component**
- Handles lane merges and splits
- Smooth blending between different lane counts
- Essential for highway on-ramps, lane reductions

### Spatial Query System

LaneGraph uses a two-level Bounding Volume Hierarchy (BVH) for lightning-fast queries:
- Component-level BVH for broad-phase filtering
- Lane-level queries within relevant components
- Designed to handle tens of thousands of lanes efficiently

### Lane States

Lanes can be in different operational states:
- **Open**: Normal operation
- **Closed**: Blocked/unavailable
- **AboutToClose**: Yellow light/warning state
- **Yield**: Proceed with caution
- **Custom states**: Extend for your specific needs

## Technical Overview

### Architecture

```
LaneGraphManager (Runtime API)
    │
    ├── LaneGraphRegistry (Data Storage)
    │   ├── Lanes (All lane data)
    │   ├── Components (Component metadata)
    │   └── Scene Index (Per-scene organization)
    │
    ├── LaneGraphBVHSystem (Spatial Queries)
    │   ├── Component BVH
    │   └── Lane Queries
    │
    └── LaneStateController (Traffic Control)
        └── Signal Groups
```

### Workflow

1. **Design Time**: Create components in editor, configure profiles
2. **Build Time**: Graph builder processes components into runtime data
3. **Runtime**: Query system provides fast lane lookups and pathfinding
4. **Dynamic Control**: Update lane states for traffic management

## Performance Characteristics

- **Initialization**: O(n log n) where n = number of components
- **Closest Lane Query**: O(log n + m) where m = lanes in nearby components
- **Radius Query**: O(log n + k) where k = lanes in radius
- **State Changes**: O(1) direct lane access

## Use Case Examples

### Racing Game
Create multi-lane racing tracks with pit lanes, shortcuts, and dynamic track sections that open/close during races.

### Traffic Simulation
Build realistic city road networks with intersections, traffic lights, lane merges, and different vehicle types following specific lanes.

### Strategy Game
Design unit movement corridors with choke points, multiple routes, and dynamic path availability based on game state.

### Open World Navigation
Implement vehicle AI that follows road networks, obeys traffic rules, and responds to dynamic road conditions.

## System Requirements

- Unity 2021.3 or later
- No external dependencies
- Works with all Unity rendering pipelines (Built-in, URP, HDRP)
- Compatible with IL2CPP and Mono scripting backends
- Supports all major platforms (PC, Console, Mobile, WebGL)

## What's Included

- **Runtime System**: Full lane graph management and query API
- **Editor Tools**: Visual component editors with scene view manipulation
- **Project Settings**: Centralized lane profile management
- **Graph Builder**: Automated data generation and optimization
- **Lane State Controller**: Traffic signal management system
- **Example Scenes**: Demonstration of core features and workflows
- **Complete Documentation**: Manual, API reference, and tutorials

## Next Steps

- [Getting Started Guide](getting-started.md) - Set up your first lane network
- [Architecture Overview](architecture.md) - Understand the system design
- [Basic Setup Tutorial](tutorials/basic-setup.md) - Create your first path

---

Ready to dive in? Start with the [Getting Started](getting-started.md) guide!
