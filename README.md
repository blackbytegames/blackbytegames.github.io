# Animecs

> High-performance GPU animation system for Unity DOTS

Animecs brings your characters to life using compute shader-based skeletal animation, giving you the freedom to animate thousands of entities without breaking a sweat.

## What is Animecs?

Think of Animecs as your animation powerhouse for DOTS. Instead of Unity's traditional Mecanim, you get:

- **GPU-accelerated skinning** - Your bones live on the GPU, not the CPU
- **Smart LOD system** - Far away characters update less frequently (they won't notice)
- **Blend trees that actually blend** - 1D, 2D directional, 2D cartesian, and direct blend trees
- **Blend shapes support** - Facial animations work out of the box
- **Event system** - Footsteps, weapon trails, whatever you need

## Why Animecs?

Traditional Unity animation is great until you want 500 characters on screen. Then your frame rate tanks.

Animecs was built from day one for crowds, armies, and anything else that needs to move at scale. The entire system runs in Burst-compiled jobs with compute shader deformation.

**Performance you'll actually notice:**
- Thousands of animated characters per frame
- Automatic LOD reduces updates for distant entities
- Shared bone structures mean less memory usage
- Compute shader deformation offloads work from CPU to GPU

## Quick Start
```bash
# Install via Package Manager
# Window > Package Manager > Add package from git URL
https://github.com/yourusername/animecs.git