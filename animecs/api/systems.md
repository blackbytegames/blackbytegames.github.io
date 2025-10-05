# Systems

ECS systems that drive Animecs animation updates.

## System Group

### AnimecsSystemGroup

Updates in `SimulationSystemGroup` (order last). Contains all Animecs animation systems.

## Core Systems

### AnimecsLODSystem

Calculates distance from entities to camera and assigns LOD levels based on configured thresholds. Sets update frequency per entity.

**Requires:** `AnimecsCameraTag` component on camera entity

### AnimecsSkinInitializationSystem

Links renderer entities to their state machine root. Runs once per entity during initialization.

### AnimecsStateProgressionSystem

Advances animation time based on delta time, state speed, and LOD update frequency. Handles looping and time wrapping.

### AnimecsStateTransitionSystem

Evaluates transition conditions from Animator Controller and initiates state changes. Processes parameter-based transitions and explicit Play() calls.

### AnimecsBlendTreeSystem

Calculates motion weights for blend trees based on parameter values. Supports 1D, 2D directional, 2D cartesian, and direct blend types.

### AnimecsEventSystem

Detects when animation crosses event frames and populates triggered event buffer for consumption.

### AnimecsSkinDeformationSystem

Samples pre-baked bone matrices and writes to SkinMatrix buffer for GPU skinning. Handles single state playback, transitions, and blend tree blending.

### AnimecsBlendShapeSystem

Applies blend shape weights from baked animation data or runtime overrides. Writes to BlendShapeWeight buffer.

## Support Systems

### AnimecsLODConfigInitSystem

Creates LOD configuration singleton at initialization. Waits for camera tag before creating config.

### AnimecsDebugVisualizationSystem

Collects debug information for runtime visualization overlay.

## Execution Order

Systems execute in this sequence each frame:

1. **AnimecsLODSystem** - Distance calculation, LOD assignment
2. **AnimecsSkinInitializationSystem** - Parent linking
3. **AnimecsStateProgressionSystem** - Time advancement
4. **AnimecsStateTransitionSystem** - Transition evaluation
5. **AnimecsBlendTreeSystem** - Weight calculation
6. **AnimecsEventSystem** - Event detection
7. **AnimecsSkinDeformationSystem** - Matrix sampling
8. **AnimecsBlendShapeSystem** - Weight application

---

See [Architecture](../manual/architecture.md) for detailed system flow.