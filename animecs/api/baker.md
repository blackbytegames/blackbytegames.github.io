# Bakers

Authoring components for converting Unity data to ECS during baking.

## Overview

Bakers execute in the editor during subscene baking. They read Unity components (Animator, SkinnedMeshRenderer) and create ECS entities with Animecs components.

**Baking happens once** - When prefab is placed in subscene and Unity bakes the subscene.

## State Machine Baker

### AnimecsStateMachineBaker

Converts Animator Controller to state machine blob asset. Place on prefab root GameObject.

**Inspector Fields:**
- `States` - Extracted animation states
- `AnyStateTransitions` - Any-state transitions
- `Parameters` - Animator parameters
- `DefaultStateIndex` - Starting state
- `EnableLOD` - Enable automatic LOD

**Created During Baking:**
- `AnimecsTag` - Identifies state machine entity
- `AnimecsStateData` - Runtime state tracking
- `AnimecsStateMachineBlob` - State machine blob reference
- `AnimecsParameters` - Parameter blob reference
- `AnimecsLOD` - LOD configuration
- Parameter buffers (Float, Int, Bool, Trigger)

**Usage:**

Automatically added when using **Right-click → Create → Animecs → Bake Animecs**.

## Skin Matrix Baker

### AnimecsSkinMatrixBaker

Bakes bone matrix data for SkinnedMeshRenderer. Automatically added to each renderer during baking.

**Inspector Fields:**
- `BoneStructure` - Pre-baked bone matrices

**Created During Baking:**
- `AnimecsSkinMatrixTag` - Identifies renderer entity
- `AnimecsBoneMatrix` - Bone blob reference
- `SkinMatrix` buffer - GPU-bound bone transforms

**Bone Structure:**
- `BoneCount` - Number of bones in skeleton
- `HierarchyHash` - Unique identifier for bone structure
- `BakedSkinMatrices` - Pre-sampled bone transforms
- `ClipStartIndices` - Offset per animation clip

**Sharing:**

Multiple renderers with identical skeletons reference the same bone blob asset.

## Blend Shape Baker

### AnimecsBlendShapeBaker

Bakes blend shape weights from animation clips. Automatically added to renderers with blend shapes.

**Inspector Fields:**
- `EnableBlendShapes` - Enable blend shape support
- `UseRuntimeWeights` - Use manual weights instead of baked
- `BlendShapeData` - Pre-baked weight data

**Created During Baking:**
- `AnimecsBlendShapeTag` - Identifies blend shape entity
- `AnimecsBlendShapeData` - Runtime control mode
- `AnimecsBlendShapeBlob` - Weight blob reference
- `AnimecsBlendShapeWeight` buffer - Per-shape weights

**Blend Shape Data:**
- `BlendShapeNames` - Shape identifiers from mesh
- `BakedWeights` - Pre-sampled weights per frame
- `ClipWeightStartIndices` - Offset per clip

**Runtime Mode:**

Set `UseRuntimeWeights = true` to control weights manually:

```csharp
var blendShapeData = entityManager.GetComponentData<AnimecsBlendShapeData>(entity);
blendShapeData.UseRuntimeWeights = true;
entityManager.SetComponentData(entity, blendShapeData);

var weights = entityManager.GetBuffer<AnimecsBlendShapeWeight>(entity);
weights[0] = new AnimecsBlendShapeWeight { Weight = 1.0f };
```

## Debug Visualizer

### AnimecsDebuggerAuthoring

Adds debug overlay showing animation state. Optional component.

**Inspector Fields:**
- `DebugMode` - Display flags (None, States, All)
- `DebugColor` - Custom color override (zero alpha uses LOD color)

**Created During Baking:**
- `AnimecsDebugger` - Debug visualization settings

**Overlay Shows:**
- Current state name
- Target state name (during transitions)
- Animation time
- LOD level (color-coded: green=High, yellow=Medium, red=Low, gray=Off, blue=Transitioning, orange=Paused)
- Transition progress
- Active blend tree motion count

## Baking Process

### State Machine Extraction

1. Read Animator Controller from prefab
2. Extract states, transitions, parameters
3. Sample animation clips at target FPS
4. Generate blob assets for states and parameters
5. Add components to root entity

### Bone Matrix Baking

1. Identify unique bone structures (by hash)
2. For each structure:
   - Sample all animation clips
   - Extract bone transforms per frame
   - Convert to local space
   - Store in flat array
3. Create shared blob assets
4. Link renderers to bone blobs

### Blend Shape Baking

1. Check if mesh has blend shapes
2. Sample blend shape weights per clip frame
3. Store weights in flat array
4. Create blob asset
5. Add blend shape components

## Baking Configuration

### Frame Rate

Default: 30 FPS

Configurable in baker code:
```csharp
const int DefaultFrameRate = 30;
```

Higher FPS = smoother animation, larger blob assets.

### Material Creation

Baking creates compute-compatible materials in `Assets/Animecs/Materials/`.

Original material properties copied to new shader.

## Validation

Bakers validate prefab during baking:

**Required:**
- Animator component with Animator Controller
- At least one SkinnedMeshRenderer
- Mesh with bone weights and bind poses
- Animation clips in Animator Controller

**Warnings:**
- No blend shapes detected (optional)
- Nested blend trees (not supported)
- Multiple animator layers (only base layer used)

## Custom Bakers

Extend Animecs with custom authoring:

```csharp
public class CustomAnimationAuthoring : MonoBehaviour
{
    public float CustomSpeed = 1.0f;

    class Baker : Baker<CustomAnimationAuthoring>
    {
        public override void Bake(CustomAnimationAuthoring authoring)
        {
            var entity = GetEntity(TransformUsageFlags.Dynamic);
            AddComponent(entity, new CustomAnimationData
            {
                Speed = authoring.CustomSpeed
            });
        }
    }
}
```

## Performance Considerations

### Baking Time

Large animation libraries take minutes to bake:

- 10 states, 5 clips: ~10 seconds
- 50 states, 20 clips: ~60 seconds
- 100 states, 50 clips: ~5 minutes

Bone matrix sampling is the bottleneck.

### Blob Asset Size

Pre-sampled data uses more memory than compressed curves:

**Example:**
- 30 bones, 100 frames, 10 clips
- 30 × 100 × 10 × 48 bytes = ~1.4 MB per skeleton

Multiple characters share assets if using same rig.

### Optimization

**Reduce frame count:** Lower target FPS in animations.

**Share skeletons:** Reuse same prefab for character variants

**Trim clips:** Remove unused animations from Animator Controller

---