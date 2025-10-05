# Getting Started

This guide walks you through installing Animecs and baking your first animated character. If you have a character with an Animator Controller, you'll have it running in DOTS in about 5 minutes.

## Prerequisites

Before using Animecs, make sure you have:

- **Unity 2022.3 or newer**
- **Entities package 1.0.16+** (installed automatically with Animecs)
- **Entities Graphics 1.0.16+** (installed automatically with Animecs)
- **URP or HDRP** (required by Entities Graphics)

> **Tip:** Use the latest stable versions of Entities and Entities Graphics. They contain important fixes and performance improvements that could benefit Animecs.

## Installation

### Via Unity Asset Store

1. Download Animecs from the [Asset Store](https://assetstore.unity.com)
2. Import into your project
3. Dependencies install automatically

## Prepare Your Character

Animecs works with standard Unity animated characters. You need:

### 1. A Rigged Character Model

Your character should have:
- A skinned mesh with bone weights
- A skeleton hierarchy (bones as GameObjects)
- Bind poses properly set up

This is standard for any FBX character you'd use with Unity's Animator.

### 2. An Animator Controller

Create or assign an Animator Controller with:
- At least one animation state
- Animation clips assigned to states
- Parameters (if using transitions)
- Blend trees (if needed)

**Important:** Set the Animator's culling mode to `Always Animate`. This ensures animations sample correctly during baking.

### 3. Model Import Settings

In your character's import settings:

**Do NOT check "Optimize Game Objects"**  
Animecs needs access to all bone GameObjects during baking. Optimized hierarchies hide this data.

**Animation Type can be anything**  
Generic, Humanoid, Legacy—doesn't matter. Animecs reads the Animator Controller, not the Avatar.

## Bake Your Character

This is the entire conversion process:

1. **Select your character prefab** in the Project window (must be a prefab, not a scene instance)
2. **Right-click → Create → Animecs → Bake Animecs**
3. A configuration window appears showing what will be baked:
   - Number of states found
   - Number of transitions
   - Number of parameters
   - SkinnedMeshRenderers detected
4. **Click "Create Animecs Prefab"**
5. Done.

Animecs creates a new prefab with `_Animecs` suffix. This prefab contains:
- **AnimecsStateMachineBaker** - Your converted state machine
- **AnimecsSkinMatrixBaker** - Per-renderer bone data (auto-added)
- **AnimecsBlendShapeBaker** - Blend shape support if detected (auto-added)

All components are added automatically. You don't manually configure anything.

## Place in Subscene

To use your baked character:

1. Create an **Entities Subscene** (`GameObject > Scene > Sub Scene`)
2. Open the subscene
3. **Drag your `_Animecs` prefab** into the subscene
4. Save the subscene

When you enter Play mode, Unity bakes the subscene and your character animates using pure DOTS.

## Verify It Works

Enter Play mode and check:

- Your character appears in the scene
- Animation plays automatically (the default state)
- No errors in the Console

If you see the character but no animation, check:
- The prefab is actually in a subscene (not the main scene)
- The subscene is set to "Auto Load" mode
- Your Animator Controller has at least one state with a clip

## Optional: Add Debug Visualization

Want to see what's happening at runtime?

1. Select your baked prefab
2. Add component: `AnimecsDebuggerAuthoring`
3. Enable "Show States"

In Play mode, you'll see:
- Current state name
- Animation time
- LOD level
- Transition progress (if transitioning)
- Active blend tree motions (if using blend trees)

This appears as an overlay above each character. Useful for debugging state transitions and LOD behavior.

## Pro Tip: Finish Animator Work First

**Animecs trades flexibility for performance.** Everything is baked into blob assets at authoring time. This means:

✅ **Fast runtime** - No reflection, no string lookups, pure data  
❌ **No runtime editing** - Can't add states or change transitions after baking

**Best practice:**  
Finish all your Animator Controller work (states, transitions, blend trees, parameters) before baking. If you need to change something later, update the original Animator Controller and re-bake.
Baking is fast (seconds for most characters), so iteration isn't painful. But plan your state machine structure before baking hundreds of prefab variants.

## What Got Baked?

When you bake a character, Animecs creates:

### Blob Assets
- **State machine structure** - States, transitions, conditions
- **Animation clips** - Frame counts, FPS, loop settings
- **Blend trees** - Motion weights, thresholds, parameters
- **Bone matrices** - Pre-sampled transforms for every frame of every clip
- **Blend shape weights** - Per-frame blend shape values (if present)

### Materials
- **Compute deformation materials** - Copies your materials with compute-compatible shaders
- Stored in `Assets/Animecs/Materials/`

### Component Data
- **State machine root** - AnimecsStateMachineBaker on prefab root
- **Per-renderer bakers** - AnimecsSkinMatrixBaker on each SkinnedMeshRenderer
- **Blend shape bakers** - AnimecsBlendShapeBaker on renderers with blend shapes

All of this happens automatically. You just click "Bake."

## Common Issues

### "No Animator or AnimatorController found" 
Your prefab needs an Animator component with an Animator Controller assigned.

### "No SkinnedMeshRenderer found"
Animecs requires at least one SkinnedMeshRenderer. Static meshes won't work.

### "Mesh has no bone weights"
Your mesh needs proper skinning. Re-import your model or check skinning in your 3D software.

### Character appears but doesn't animate
Make sure the prefab is in a subscene, not the main scene. DOTS baking only works in subscenes.

---

**You're ready!** Your character is now animating in pure DOTS. Check the [tutorials](tutorials/basic-setup.md) for more features.