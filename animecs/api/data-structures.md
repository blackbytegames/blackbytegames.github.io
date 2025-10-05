# Data Structures

Blob assets and runtime data structures for Animecs.

## Blob Assets

Blob assets contain baked animation data. Created during authoring, read-only at runtime.

### AnimecsStateMachineBlobAsset

State machine definition with all states and transitions.

**Structure:**
- `States[]` - Array of state definitions
- `AnyStateTransitions[]` - Transitions active from any state

**Access:**
```csharp
var stateMachineBlob = entityManager.GetComponentData<AnimecsStateMachineBlob>(entity);
ref var stateMachine = ref stateMachineBlob.StateMachineBlob.Value;

// Read state data
ref var state = ref stateMachine.States[stateIndex];
var stateName = state.StateName;
var speed = state.Speed;
```

### AnimecsStateBlobData

Individual state configuration.

**Fields:**
- `StateName` - State identifier (FixedString64Bytes)
- `Speed` - Playback speed multiplier
- `CycleOffset` - Normalized start offset (0-1)
- `Motion` - Blend tree definition
- `Transitions[]` - Outgoing transitions
- `Events[]` - Animation events

### AnimecsStateTreeBlobData

Blend tree configuration for state motion.

**Fields:**
- `TreeType` - Blend tree type (1D, 2D, Direct)
- `BlendParameterHash` - Primary blend parameter
- `BlendParameterHashY` - Secondary blend parameter (2D)
- `Motions[]` - Animation clips with blend data

**Tree Types:**
- `Single` - No blending
- `BlendTree1D` - Linear blend
- `BlendTree2DSimpleDirectional` - Directional 2D
- `BlendTree2DFreeformDirectional` - Freeform directional
- `BlendTree2DFreeformCartesian` - Position-based
- `BlendTreeDirect` - Manual weight control

### AnimecsMotionBlobData

Single motion within blend tree.

**Fields:**
- `Clip` - Animation clip data
- `Threshold` - Blend threshold (1D)
- `PositionX` - X position (2D)
- `PositionY` - Y position (2D)
- `TimeScale` - Playback speed multiplier
- `DirectBlendParameterHash` - Direct blend weight parameter

### AnimecsClipBlobData

Animation clip metadata.

**Fields:**
- `ClipName` - Clip identifier
- `Fps` - Target frame rate
- `FrameCount` - Total frames
- `Loop` - Loop flag

### AnimecsStateTransitionBlobData

Transition definition between states.

**Fields:**
- `TargetStateIndex` - Destination state
- `HasExitTime` - Wait for animation completion
- `StartTime` - Normalized exit time (0-1)
- `Duration` - Blend duration in seconds
- `BlendStrength` - Curve strength (0-1)
- `Conditions[]` - Evaluation conditions

### AnimecsTransitionConditionData

Condition for transition evaluation.

**Fields:**
- `ParameterHash` - Parameter identifier
- `Mode` - Comparison mode
- `Threshold` - Comparison value

**Condition Modes:**
- `Greater` - Value > threshold
- `Less` - Value < threshold
- `Equal` - Value == threshold
- `NotEqual` - Value != threshold
- `If` - Bool/trigger is true
- `IfNot` - Bool/trigger is false

### AnimecsAnimationEventData

Animation event at specific frame.

**Fields:**
- `FrameIndex` - Trigger frame
- `EventName` - Function identifier
- `FloatParameter` - Custom float data
- `IntParameter` - Custom int data

### AnimecsBoneBlobAsset

Pre-baked bone matrix data for skeleton.

**Structure:**
- `SkinMatrices[]` - Flat array of bone transforms
- `ClipMatrixStartIndices[]` - Clip offset indices
- `TotalFrameCount` - Total frames across all clips

**Access Pattern:**
```csharp
int clipStartIndex = boneBlob.Value.ClipMatrixStartIndices[clipIndex];
int matrixIndex = clipStartIndex + (frame * boneCount) + boneIndex;
float3x4 boneMatrix = boneBlob.Value.SkinMatrices[matrixIndex];
```

### AnimecsBlendShapeBlobAsset

Pre-baked blend shape weight data.

**Structure:**
- `BlendShapeCount` - Number of blend shapes
- `BlendShapeNames[]` - Shape identifiers
- `BakedWeights[]` - Per-frame weights
- `ClipWeightStartIndices[]` - Clip offset indices

## Parameter Assets

### AnimecsParameterBlobAsset

Parameter metadata and default values.

**Structure:**
- `Parameters[]` - Sorted parameter entries
- `FloatValues[]` - Default float values
- `IntValues[]` - Default int values
- `BoolValues[]` - Default bool values
- `TriggerValues[]` - Default trigger values

**Access:**
```csharp
var parameters = entityManager.GetComponentData<AnimecsParameters>(entity);
ref var paramBlob = ref parameters.ParameterBlob.Value;

// Parameters sorted by hash for binary search
```

### AnimecsParameterEntry

Parameter metadata entry.

**Fields:**
- `NameHash` - Parameter identifier
- `Type` - Parameter type (Float, Int, Bool, Trigger)
- `ValueIndex` - Index into type-specific value array

## Runtime Data

### AnimecsBlendData

State transition blend progress.

**Fields:**
- `BlendProgress` - Current progress (0-1)
- `BlendDuration` - Total duration in seconds
- `BlendCurveType` - Interpolation curve

### AnimecsBlendTreeRuntimeData

Cached blend tree evaluation.

**Fields:**
- `Weight0` through `Weight7` - Motion weights (max 8)
- `ActiveMotionCount` - Active motion count
- `ParamValueX` - Cached X parameter
- `ParamValueY` - Cached Y parameter

**Usage:**
```csharp
var stateData = entityManager.GetComponentData<AnimecsStateData>(entity);
float firstMotionWeight = stateData.BlendTreeData.Weight0;
byte activeCount = stateData.BlendTreeData.ActiveMotionCount;
```

## Enums

### AnimecsParameterType

```csharp
public enum AnimecsParameterType : byte
{
    Float,
    Int,
    Bool,
    Trigger
}
```

### AnimecsConditionMode

```csharp
public enum AnimecsConditionMode : byte
{
    Greater,
    Less,
    Equal,
    NotEqual,
    If,
    IfNot
}
```

### AnimecsLODLevel

```csharp
public enum AnimecsLODLevel : byte
{
    High,    // Every frame
    Medium,  // Every 2 frames (default)
    Low,     // Every 8 frames (default)
    Off      // No updates
}
```

## Memory Layout

### Bone Matrix Storage

Matrices stored as flat array:
```
[Clip0_Frame0_Bone0, Clip0_Frame0_Bone1, ..., Clip0_Frame1_Bone0, ...]
```

Index calculation:
```csharp
int index = clipStartIndex + (frame * boneCount) + boneIndex;
```

### Blend Shape Storage

Weights stored as flat array:
```
[Clip0_Frame0_Shape0, Clip0_Frame0_Shape1, ..., Clip0_Frame1_Shape0, ...]
```

Index calculation:
```csharp
int index = clipStartIndex + (frame * shapeCount) + shapeIndex;
```

## Blob Asset Sharing

Entities with identical skeletons share bone blob assets:

```csharp
// Same skeleton = same bone blob reference
Character_A.BoneBlob == Character_B.BoneBlob // true if same rig
```

Hash calculated from:
- Bone count
- Bone hierarchy
- Bind poses
- Animation clip set

---

**See Also:**

- [Components →](components.md) - Component definitions
- [AnimecsController →](controller.md) - Runtime API
- [Bakers →](baker.md) - Authoring components