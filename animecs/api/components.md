# Components

ECS components for the Animecs animation system.

## Core Components

### AnimecsTag

Identifies entities using Animecs state machine. Enableable component.

### AnimecsStateData

Runtime animation state containing current/target state indices, animation time, blend progress, and state flags.

**State Flags:**
- Paused
- Force frame update
- Keyframe-only mode
- Blending between states
- Using blend tree

### AnimecsStateMachineBlob

Reference to baked state machine blob asset containing states, transitions, and parameters.

### AnimecsParameters

Reference to parameter blob asset with parameter metadata and default values.

### Parameter Buffers

Per-entity storage for parameter values:
- `AnimecsFloatParameter` - Float values
- `AnimecsIntParameter` - Int values
- `AnimecsBoolParameter` - Bool values
- `AnimecsTriggerParameter` - Trigger values

## Renderer Components

### AnimecsSkinMatrixTag

Identifies renderer entities with GPU skinning enabled. Enableable component.

### AnimecsBoneMatrix

Bone structure data and reference to pre-baked bone matrix blob.

### AnimecsStateMachineParent

Links renderer entity to its state machine root entity.

## LOD Components

### AnimecsCameraManaged

Managed component holding Unity Camera reference for LOD calculations. Optional singleton.

**Fields:**
- `Camera` - Unity Camera reference

### AnimecsLOD

LOD state containing current level, update frequency, frame offset, and distance to camera.

### AnimecsLODConfig

Global LOD configuration with distance thresholds, update frequencies, and temporal distribution settings. Singleton component.

## Blend Shape Components

### AnimecsBlendShapeTag

Identifies entities with blend shape support. Enableable component.

### AnimecsBlendShapeData

Blend shape control mode (baked or runtime weights).

### AnimecsBlendShapeBlob

Reference to pre-baked blend shape weight data.

### AnimecsBlendShapeWeight

Buffer element for per-blend-shape weight values (0-1 range).

## Event Components

### AnimecsEventReceiverTag

Enables animation event detection for entity. Enableable component.

### AnimecsEventReceiver

Event detection state tracking previous frame index for crossing detection.

### AnimecsTriggeredEvent

Buffer element for fired animation events. Contains event name, state index, frame index, trigger time, and custom parameters.

## Debug Components

### AnimecsDebugger

Debug visualization settings including display mode and color override. Enableable component.

**Fields:**
- `DebugMode` - Flags for what to display (States, All)
- `DebugColor` - Custom color override. If alpha is zero, uses automatic LOD colors: High (Green), Medium (Yellow), Low (Red), Off (Gray), Transitioning (Blue), Paused (Orange)

---