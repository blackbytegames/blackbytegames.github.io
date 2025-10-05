# Blend Shapes

Blend shapes (also called morph targets or shape keys) animate mesh deformation. Common uses include facial expressions, lip sync, and muscle bulging.

Animecs supports two modes: **baked weights** (pre-sampled from animations) and **runtime weights** (manual control).

## Baked Blend Shapes

Default mode. Animecs samples blend shape weights from your animation clips during baking and plays them back at runtime.

### Setup

If your mesh has blend shapes and your animation clips animate them, baking handles everything automatically:

1. Ensure your SkinnedMeshRenderer has blend shapes
2. Animation clips modify blend shape weights
3. Bake the prefab (`Right-click → Create → Animecs → Bake Animecs`)

The baker adds `AnimecsBlendShapeBaker` to renderers with blend shapes.

### Verification

Check the baked prefab:

```
MyCharacter_Animecs
  └─ Body (SkinnedMeshRenderer)
      ├─ AnimecsSkinMatrixBaker
      └─ AnimecsBlendShapeBaker  ← Added automatically
```

Inspect `AnimecsBlendShapeBaker`:
- `EnableBlendShapes`: true
- `UseRuntimeWeights`: false (baked mode)
- `BlendShapeData`: Contains baked weights

### Runtime Behavior

Blend shape weights update automatically with animation state:

```csharp
// Play talking animation
AnimecsController.Play(ecb, index, entity, "Talk", transitionDuration: 0.2f);

// Blend shapes (jaw, lips) animate automatically
// No manual control needed
```

Weights are interpolated between frames just like bone transforms.

## Runtime Blend Shapes

Override baked weights with manual control. Useful for procedural facial animation or lip sync.

### Enabling Runtime Mode

Edit the baked prefab:

1. Select renderer with `AnimecsBlendShapeBaker`
2. Enable `UseRuntimeWeights`

Or set at runtime:

```csharp
var blendShapeData = entityManager.GetComponentData<AnimecsBlendShapeData>(entity);
blendShapeData.UseRuntimeWeights = true;
entityManager.SetComponentData(entity, blendShapeData);
```

### Setting Weights

Blend shape weights are stored in the `AnimecsBlendShapeWeight` buffer:

```csharp
[BurstCompile]
partial struct FacialExpressionJob : IJobEntity
{
    void Execute(ref DynamicBuffer<AnimecsBlendShapeWeight> weights, 
                 in FacialEmotion emotion)
    {
        // Blend shape indices (from mesh in modeling software)
        const int SmileIndex = 0;
        const int FrownIndex = 1;
        const int EyebrowUpIndex = 2;

        if (emotion.Type == EmotionType.Happy)
        {
            weights[SmileIndex] = new AnimecsBlendShapeWeight { Weight = 1.0f };
            weights[FrownIndex] = new AnimecsBlendShapeWeight { Weight = 0.0f };
            weights[EyebrowUpIndex] = new AnimecsBlendShapeWeight { Weight = 0.5f };
        }
        else if (emotion.Type == EmotionType.Sad)
        {
            weights[SmileIndex] = new AnimecsBlendShapeWeight { Weight = 0.0f };
            weights[FrownIndex] = new AnimecsBlendShapeWeight { Weight = 0.8f };
            weights[EyebrowUpIndex] = new AnimecsBlendShapeWeight { Weight = 0.0f };
        }
    }
}
```

Weight range: **0.0 to 1.0** (0% to 100% influence)

### Finding Blend Shape Indices

Blend shape order matches your mesh in Unity:

```csharp
// In editor, inspect mesh:
// Blend Shapes
//   0: Smile
//   1: Frown
//   2: EyebrowUp
//   3: JawOpen

const int SmileIndex = 0;
const int JawOpenIndex = 3;
```

Or query at runtime (managed code only):

```csharp
var skinnedRenderer = GetComponent<SkinnedMeshRenderer>();
var mesh = skinnedRenderer.sharedMesh;

for (int i = 0; i < mesh.blendShapeCount; i++)
{
    Debug.Log($"{i}: {mesh.GetBlendShapeName(i)}");
}
```

## Combining Modes

Use baked weights for body animations, runtime weights for facial expressions:

### Separate Renderers

```
Character
  ├─ Body (baked blend shapes for breathing)
  │   └─ UseRuntimeWeights: false
  └─ Head (runtime blend shapes for lip sync)
      └─ UseRuntimeWeights: true
```

Each renderer can have different blend shape modes.

### Hybrid Approach

One renderer, switch modes based on gameplay:

```csharp
void StartCutscene(Entity entity, EntityManager entityManager)
{
    // Use runtime control for lip sync during dialogue
    var blendShapeData = entityManager.GetComponentData<AnimecsBlendShapeData>(entity);
    blendShapeData.UseRuntimeWeights = true;
    entityManager.SetComponentData(entity, blendShapeData);
}

void EndCutscene(Entity entity, EntityManager entityManager)
{
    // Return to baked animation
    var blendShapeData = entityManager.GetComponentData<AnimecsBlendShapeData>(entity);
    blendShapeData.UseRuntimeWeights = false;
    entityManager.SetComponentData(entity, blendShapeData);
}
```

## Lip Sync Example

Procedural lip sync using phoneme-based blend shapes:

```csharp
[BurstCompile]
public partial struct LipSyncSystem : ISystem
{
    public void OnUpdate(ref SystemState state)
    {
        foreach (var (weights, lipSync) in 
            SystemAPI.Query<DynamicBuffer<AnimecsBlendShapeWeight>, RefRO<LipSyncData>>())
        {
            // Phoneme blend shapes (common setup)
            const int JawOpenIndex = 0;
            const int LipPuckerIndex = 1;
            const int LipWideIndex = 2;

            float time = (float)SystemAPI.Time.ElapsedTime;
            int currentPhoneme = GetPhonemeAtTime(lipSync.ValueRO, time);

            // Map phoneme to blend shape weights
            switch (currentPhoneme)
            {
                case 'A': // "ah"
                    weights[JawOpenIndex] = new AnimecsBlendShapeWeight { Weight = 0.8f };
                    weights[LipPuckerIndex] = new AnimecsBlendShapeWeight { Weight = 0.0f };
                    weights[LipWideIndex] = new AnimecsBlendShapeWeight { Weight = 0.3f };
                    break;

                case 'O': // "oh"
                    weights[JawOpenIndex] = new AnimecsBlendShapeWeight { Weight = 0.5f };
                    weights[LipPuckerIndex] = new AnimecsBlendShapeWeight { Weight = 0.9f };
                    weights[LipWideIndex] = new AnimecsBlendShapeWeight { Weight = 0.0f };
                    break;

                case 'E': // "ee"
                    weights[JawOpenIndex] = new AnimecsBlendShapeWeight { Weight = 0.2f };
                    weights[LipPuckerIndex] = new AnimecsBlendShapeWeight { Weight = 0.0f };
                    weights[LipWideIndex] = new AnimecsBlendShapeWeight { Weight = 1.0f };
                    break;

                default: // Neutral
                    weights[JawOpenIndex] = new AnimecsBlendShapeWeight { Weight = 0.0f };
                    weights[LipPuckerIndex] = new AnimecsBlendShapeWeight { Weight = 0.0f };
                    weights[LipWideIndex] = new AnimecsBlendShapeWeight { Weight = 0.0f };
                    break;
            }
        }
    }

    static int GetPhonemeAtTime(in LipSyncData data, float time)
    {
        // Your phoneme timing logic
        return 'A';
    }
}
```

## Performance Considerations

### Blend Shape Cost

Blend shapes are deformed on GPU via compute shader. Cost depends on:
- Number of blend shapes in mesh
- Vertex count
- Number of active blend shapes

**Typical impact:** Negligible for 10-20 blend shapes on 5k-vertex mesh.

### Optimization Tips

**Reduce blend shape count** - Remove unused blend shapes from mesh in modeling software.

**Use LOD** - Disable blend shapes at Low LOD if barely visible:

```csharp
[BurstCompile]
partial struct BlendShapeLODJob : IJobEntity
{
    void Execute(ref AnimecsBlendShapeData blendShapeData, in AnimecsLOD lod)
    {
        // Disable blend shapes at low LOD
        blendShapeData.UseRuntimeWeights = lod.Level != AnimecsLODLevel.Low;

        if (lod.Level == AnimecsLODLevel.Low)
        {
            // Reset to neutral
            // (weights buffer cleared by system)
        }
    }
}
```

**Combine blend shapes** - Merge rarely-used shapes in modeling software (e.g., "SmileLeft" + "SmileRight" → "Smile").

## Debugging Blend Shapes

### Verify Baking

Check `AnimecsBlendShapeBaker` on baked prefab:

```csharp
// BlendShapeData should show:
// - BlendShapeNames: ["Smile", "Frown", ...]
// - BakedWeights: [0.0, 0.0, 0.5, ...] (per frame/clip)
// - ClipWeightStartIndices: [0, 150, 300, ...] (clip offsets)
```

Empty arrays mean no blend shape animation in clips.

### Runtime Weight Inspection

```csharp
var weights = entityManager.GetBuffer<AnimecsBlendShapeWeight>(entity);
for (int i = 0; i < weights.Length; i++)
{
    Debug.Log($"BlendShape {i}: {weights[i].Weight}");
}
```

### Visual Debugging

```csharp
// Force specific blend shape for testing
var weights = entityManager.GetBuffer<AnimecsBlendShapeWeight>(entity);
weights[0] = new AnimecsBlendShapeWeight { Weight = 1.0f }; // Max smile
```

If mesh doesn't deform, check:
- Mesh actually has blend shapes
- `AnimecsBlendShapeBaker` present
- `UseRuntimeWeights` enabled if setting manually

## Common Issues

**Blend shapes don't animate** - Animation clips don't modify blend shape weights. Check clips in Animation window.

**Weights reset to zero** - `UseRuntimeWeights` is false. Baked mode overrides manual weights.

**Mesh explodes/artifacts** - Weight values outside 0-1 range. Clamp to valid range.

**Some blend shapes missing** - Mesh imported without blend shapes. Re-import with "Import BlendShapes" enabled.

---

**Congratulations!** You've completed all Animecs tutorials. You now know how to:
- Control animations at runtime
- Use state machines and transitions
- Set up blend trees for locomotion
- Optimize with LOD
- Animate blend shapes