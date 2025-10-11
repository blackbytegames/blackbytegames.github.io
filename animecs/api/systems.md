# Systems

ECS systems that drive Animecs animation updates.

## System Group

### AnimecsSystemGroup

Updates in `SimulationSystemGroup` (order last). Contains all Animecs animation systems.

## Core Systems

### AnimecsCameraSystem

Updates camera position and view-projection matrix from `AnimecsCameraManaged` or Camera.main. Runs before `AnimecsLODSystem`.

**Requires:** `AnimecsCameraManaged` managed component (optional, falls back to Camera.main)

### AnimecsLODSystem

Calculates distance from entities to camera and assigns LOD levels based on configured thresholds. Performs frustum culling to set Off state for out-of-view entities.

**Requires:** `AnimecsLODConfig` singleton (created automatically)

### AnimecsSkinInitializationSystem

Links renderer entities to their state machine root. Runs once per entity during initialization. Runs once in `AnimecsInitializationSystemGroup`.

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

Creates LOD configuration singleton at initialization with default values. Runs once in `AnimecsInitializationSystemGroup`.

### AnimecsDebugVisualizationSystem

Collects debug information for runtime visualization overlay.

## Execution Order

Systems execute in this sequence each frame:

1. **AnimecsCameraSystem** - Camera data update
2. **AnimecsLODSystem** - Distance calculation, frustum culling, LOD assignment
3. **AnimecsSkinInitializationSystem** - Parent linking
4. **AnimecsStateProgressionSystem** - Time advancement
5. **AnimecsStateTransitionSystem** - Transition evaluation
6. **AnimecsBlendTreeSystem** - Weight calculation
7. **AnimecsEventSystem** - Event detection
8. **AnimecsSkinDeformationSystem** - Matrix sampling
9. **AnimecsBlendShapeSystem** - Weight application

---