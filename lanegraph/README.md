# LaneGraph Documentation

Welcome to the LaneGraph documentation! LaneGraph is a lightweight lane-based navigation system designed for pathfinding and traffic simulation in Unity.

## What is LaneGraph?

LaneGraph provides a comprehensive solution for creating and managing lane-based navigation networks in Unity. Whether you're building a racing game, traffic simulation, or any application requiring structured pathways, LaneGraph offers the tools you need.

### Key Features

- **Lane-Based Pathfinding**: Create complex road networks with multiple lane types and directions
- **Three Component Types**: Paths, Intersections, and Transitions for maximum flexibility
- **Traffic Control**: Built-in lane state controller for signal management
- **Optimized Spatial Queries**: High-performance BVH system for efficient lane searches
- **Visual Editor Tools**: Intuitive scene view tools with real-time visualization
- **Flexible Lane Profiles**: Configurable lane widths, directions, tags, and speeds
- **Automatic Snapping**: Components automatically connect for seamless networks

### Use Cases

- **Racing Games**: Create racing tracks with multiple lanes and pit lanes
- **Traffic Simulation**: Build realistic road networks with intersections and merges
- **Strategy Games**: Design movement lanes for units and AI pathfinding
- **Open World Games**: Implement road networks for vehicle navigation
- **Logistics Simulations**: Create warehouse paths and delivery routes

## Quick Start

1. **Install LaneGraph** package into your Unity project
2. **Create Lane Profiles** via Edit > Project Settings > Lane Graph
3. **Add Lane Components** to your scene (GameObject > Lane > ...)
4. **Build the Lane Graph** via Tools > LaneGraph > Build LaneGraph
5. **Query at Runtime** using the `LaneGraphManager` API

## Documentation Structure

This documentation is organized into three main sections:

### Manual
Comprehensive guides covering all aspects of LaneGraph, from basic setup to advanced techniques. Includes step-by-step tutorials for common workflows.

### Scripting API
Detailed API reference for all public classes, methods, and properties. Essential for runtime integration and custom tools.

### Resources
Additional resources including changelog, license information, and community links.

## Getting Help

- **Email Support**: sridogames@gmail.com
- **GitHub**: [Blackbyte Games](https://github.com/blackbytegames)
- **Asset Store**: View on [Unity Asset Store](https://assetstore.unity.com)

## System Requirements

- **Unity Version**: 2021.3 or later
- **Scripting Backend**: IL2CPP or Mono
- **Platforms**: Windows, Mac, Linux, iOS, Android, WebGL

---

Ready to get started? Check out the [Getting Started](manual/getting-started.md) guide or jump into the [tutorials](manual/tutorials/basic-setup.md)!
