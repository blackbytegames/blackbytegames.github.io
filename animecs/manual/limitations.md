# Limitations

Animecs bakes animations to GPU-ready data during authoring. This trades flexibility for performance. Here's what isn't supported:

## Feature Support

| Feature | Supported | Reason |
|---------|-----------|--------|
| **Multi Layers** | No | Only base layer supported |
| **Root Motion** | No | |
| **Humanoid Retargeting** | No | |
| **IK / Constraints** | No | Per-frame solving, can't be pre-baked |
| **Avatar Masks** | No | Per-bone blending adds CPU cost |
| **Runtime Controller Swap** | No | State machines baked into blob assets |
| **Nested Blend Trees** | No | |
| **Curve Bindings** | No | Only bone transforms and blend shapes sampled |
| **Sub-State Machines** | No | States flattened during baking |

## Blend Tree Limits

- **Maximum 8 motions** per blend tree
- **No nested blend trees**
- 2D blend trees supported (Simple/Freeform Directional, Freeform Cartesian, Direct)

## Data Characteristics

**Memory:** Pre-baked frames consume more memory than compressed curves  
**Baking:** Large animation sets take minutes to process  
**Runtime:** Zero CPU decompression, all data GPU-ready

> **Pro Tip:** Most limitations can be solved in your modeling software. Need multiple animation variations? Bake them as separate clips. Need complex blending? Author blend trees in Unity's Animator Controller. Animecs focuses on maximizing runtime performance by offloading everything possible to the GPU.

## Workarounds

### Custom IK

Add system after deformation:

```csharp
[UpdateInGroup(typeof(AnimecsSystemGroup))]
[UpdateAfter(typeof(AnimecsSkinDeformationSystem))]
public partial struct CustomIKSystem : ISystem
{
    // Modify SkinMatrix buffer directly
}
```

### Root Motion

Extract movement from animation time:

```csharp
var stateData = entityManager.GetComponentData<AnimecsStateData>(entity);
float movement = CalculateRootMotion(stateData.CurrentStateIndex, deltaTime);
```

### Multiple State Machines

Use separate entities for independent animation needs:

```
Character
  ├─ Body (locomotion state machine)
  └─ Head (facial state machine)
```

## When to Use Animecs

**Good fit:**
- Crowds and large character counts
- Performance-critical scenarios
- Pre-defined animation sets
- Single-layer state machines

**Poor fit:**
- Heavy IK/constraint usage
- Fully procedural animation
- Runtime animation modification
- Humanoid retargeting required

---

**Questions?** Contact [support](mailto:sridogames@gmail.com)