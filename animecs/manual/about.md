# About Animecs

Animecs is a high-performance GPU animation system built specifically for Unity DOTS. It takes your existing Animator Controllers, converts them to DOTS-compatible data structures, and animates thousands of characters using compute shader skinning and Burst-compiled jobs.

If you've tried animating crowds in DOTS and found yourself writing state machines from scratch, Animecs solves that problem. You keep your Mecanim workflow, and Animecs handles the conversion to pure ECS.

## The Problem

Unity's DOTS architecture is great for performance, but animation support is limited. Here's what you face without Animecs:

**No State Machine Conversion**  
Mecanim doesn't work in DOTS. You either rebuild your entire animation logic manually or try to bridge GameObject animators with entities (which defeats the purpose of DOTS).

**Manual Blend Tree Implementation**  
Want smooth locomotion blending? Write your own 1D/2D blend tree math. Want directional blending? More custom code. This takes weeks before you animate a single character.

**LOD Is Your Problem**  
Characters far from camera don't need 60fps updates, but implementing distance-based LOD with temporal distribution means building another system before you even start animating.

**Compute Deformation Setup**  
Unity Entities Graphics supports GPU skinning, but you need to set up compute shaders, manage bone buffers, handle blend shapes, and write all the glue code yourself.

That's a month of work minimum. Animecs does all of this in one baking step.

## The Solution

Animecs reads your Animator Controller and converts everything to DOTS:

- **States and transitions** → Blob assets with runtime evaluation
- **Blend trees** → 1D, 2D directional, 2D cartesian, and direct blending
- **Parameters** → Float, int, bool, trigger support
- **Animation events** → Frame-accurate event detection
- **Blend shapes** → Pre-baked or runtime override
- **LOD** → Automatic distance-based update frequency

Your workflow stays the same. Animator Controller, animation clips, blend trees—all converted during baking. At runtime, Animecs uses pure ECS jobs and compute shaders to handle everything.

## Design Goals

### 1. Baking Over Runtime

We pre-compute as much as possible. Bone matrices are sampled and baked during authoring. State machines are converted to optimized blob assets. This means zero reflection, zero string lookups, and zero GameObject traversal at runtime.

**Why it matters:** Your game runs fast because the heavy work happened in the editor.

### 2. Familiar Workflow

If you know Mecanim, you know Animecs. Same states, same transitions, same blend trees. We don't reinvent animation authoring—we just make it work at scale in DOTS.

**Why it matters:** You don't rewrite your animation logic. Right-click, bake, done.

### 3. Performance by Default

LOD isn't optional. Compute shader skinning isn't a toggle. Burst compilation isn't experimental. These are enabled by default because DOTS is for performance, and Animecs respects that.

**Why it matters:** You get optimized animation without fighting configuration settings.

### 4. Explicit Behavior

When you request a 0.3 second transition, you get 0.3 seconds. When you set LOD thresholds, those exact thresholds are used. No hidden heuristics, no "smart" behavior that breaks your expectations.

**Why it matters:** Debugging is easier when systems do exactly what they say.

## What Makes It Fast

### Shared Bone Structures

Multiple characters with identical skeletons share the same baked bone matrix data. One character or one hundred—if they use the same rig, they reference the same blob asset.

**Impact:** Less memory, better cache usage, faster baking.

### Compute Shader Skinning

Vertices are transformed on the GPU via compute shaders. The CPU just updates bone transforms and dispatches compute jobs. No per-vertex CPU work, no mesh copies per instance.

**Impact:** Thousands of characters without CPU bottlenecks.

### LOD With Temporal Distribution

Distant characters update at 1/8th frequency. Updates are distributed across frames using deterministic offsets. No frame spikes when 200 characters need simultaneous updates.

**Impact:** Consistent frame times even with massive crowds.

### Blob Assets for Everything

State machines, transitions, clips, blend trees—all stored in blob assets. Fast access, safe for jobs, zero garbage collection.

**Impact:** No allocations, no cache misses, pure data-oriented performance.

### Burst-Compiled Jobs

Every system runs through Burst. State evaluation, blend tree calculation, transition checks, LOD updates—all compiled to SIMD-optimized native code.

**Impact:** 10-20x faster than managed C# for animation logic.

## Animecs vs Unity Animator

Animator is powerful and flexible. Animecs is fast and specific. Here's where they differ:

### What Animator Does Better

- **Humanoid retargeting** - One animation works on different skeletons
- **Root motion** - Built-in support for movement from animation
- **IK** - Integrated inverse kinematics
- **Extensive curve editing** - Animate literally any property
- **Runtime flexibility** - Change state machines without rebaking

### What Animecs Does Better

- **Performance at scale** - Thousands of characters per frame
- **DOTS compatibility** - Pure ECS, no GameObject overhead
- **Predictable costs** - No runtime reflection or string comparisons
- **Memory efficiency** - Shared bone data across instances
- **Explicit control** - No hidden state machine complexity

**Use Animator when:** You need maximum flexibility, complex IK, or humanoid retargeting.

**Use Animecs when:** You need performance, DOTS integration, and crowds.

## Animecs vs Other DOTS Animation Tools

There are other DOTS animation solutions on the Asset Store. Here's what makes Animecs different:

### Mecanim Compatibility

Animecs converts Animator Controllers directly. You don't rebuild your animation logic—you bake it. Other tools often require custom state machine authoring or code-based animation setup.

### Compute Shader Skinning

Animecs uses Unity's compute deformation system. Some tools use CPU skinning (slow) or custom GPU approaches (compatibility issues). Animecs integrates with Entities Graphics for proven, supported GPU skinning.

### Automatic LOD

LOD is built-in and enabled by default. Distance thresholds, update frequencies, and temporal distribution all work out of the box. Other solutions often leave LOD implementation to you.

### Blend Tree Support

Full support for Unity's blend tree types: 1D, 2D Simple Directional, 2D Freeform Directional, 2D Freeform Cartesian, and Direct. Some tools only support basic blending or require manual setup.

### Baking Workflow

Animecs pre-bakes bone matrices during authoring. This means zero runtime overhead for sampling animations. Other tools may sample clips at runtime, which adds CPU cost.

## Known Limitations

Animecs is built for specific use cases. Here's what it doesn't do:

### No Nested Blend Trees

Blend trees must be one level deep. A blend tree cannot contain another blend tree as a child. Keep your blending simple or split into multiple states.

### No IK Support

Inverse kinematics must be handled separately. Use a dedicated IK solution if needed. Animecs handles forward kinematics only.

### No Root Motion (Yet)

Root motion extraction is planned for a future update but not available in version 1.0. Use manual movement for now.

### No Humanoid Retargeting

Each skeleton needs its own animation clips. Generic rigs only. Humanoid avatar retargeting is not supported.

### No Runtime State Machine Editing

State machines are baked into blob assets. You can't add states or transitions at runtime. All animation logic must be defined during authoring.

### Compute Shader Required

GPU skinning requires compute shader support. Very old GPUs or platforms without compute capabilities won't work. (Basically anything from the last decade is fine.)

[Full limitations list →](limitations.md)

## Architecture Overview

Here's how Animecs works internally: