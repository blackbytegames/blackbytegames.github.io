# Samples

Animecs includes a demonstration scene showing all major features. Import it to see working examples of state machines, blend trees, LOD, events, and blend shapes.

## Installing the Sample

1. Open **Window → Package Manager**
2. Select **In Project** in the left dropdown
3. Click **Animecs** in the package list
4. Switch to the **Samples** tab
5. Click **Import** next to "Basic"

The sample imports to `Assets/Samples/Animecs/1.0.0/Basic/`.

## What's Included

The sample contains a single scene with 8 demonstration booths arranged in a circle. Each booth showcases a different Animecs feature:

### Booth 1: Basic Playback
Plays a single animation state continuously or uses a 1D blend tree (Idle → Walk → Run) controlled by a Speed parameter.

**Demonstrates:** Single state playback, blend tree evaluation, parameter-driven animation

### Booth 2: Blended Transitions
Cycles through states with smooth transitions: Idle → Walk → Run → Idle. Transitions use 0.3s blend duration.

**Demonstrates:** Parameter-based state changes with blending

### Booth 3: Instant Transitions
Same cycle as Booth 2, but with instant (0s) transitions. Shows the difference between blended and immediate state changes.

**Demonstrates:** Zero-duration transitions

### Booth 4: LOD Cycling
Manually cycles through LOD levels every 1.5 seconds while playing Run animation. Watch animation update frequency change.

**Demonstrates:** LOD system behavior (High → Medium → Low → Off)

### Booth 5: Event Detection
Plays walking animation with footstep events. Audio plays when animation crosses event frames.

**Demonstrates:** Animation events, frame-accurate detection

### Booth 6: Reserved
Placeholder booth for future features.

### Booth 7: Third Person Controller
Interactive character controller with player input. Move with WASD, hold Shift to sprint, Space to jump.

**Demonstrates:** Runtime parameter control, player input integration

**Controls:**
- **WASD:** Movement
- **Shift:** Sprint
- **Space:** Jump

### Booth 8: Debug Visualization
Shows debug overlay with state information, animation time, LOD level, and transition progress. Uses `AnimecsDebuggerAuthoring` component.

**Demonstrates:** Runtime debugging, state inspection

## Scene Setup

The sample uses a subscene (`DemoScene`) containing:
- 8 character prefabs (one per booth)
- Main camera with `AnimecsCameraTag`
- `AnimecsBoothManager` for camera control

### Camera Controls

**Arrow Keys:** Manually rotate camera between booths  
**Tab:** Toggle automatic rotation (default: off)

To enable auto-rotation by default, select `AnimecsBoothManager` in the Hierarchy and check "Auto Rotate".

## Code Structure

### Core Components

**AnimecsDemoAuthoring** - Configures booth behavior  
**AnimecsDemoData** - Runtime state (timer, current clip index, blend values)  
**AnimecsDemoSystem** - Updates all booth behaviors

### Booth Behavior Implementation

Each demo type runs different logic:

```csharp
public enum AnimecsDemoType : byte
{
    BasicPlayback,        // Plays blend tree state
    BlendedTransitions,   // Cycles with smooth blending
    InstantTransitions,   // Cycles with instant changes
    LODCycling,          // Manually changes LOD levels
    EventDetection,      // Demonstrates animation events
    ThirdPersonController, // Player input control
    DebugVisualization   // Debug overlay display
}
```

Check `AnimecsDemoSystem.cs` to see how each type is implemented.

## Learning from the Sample

### Parameter Control

Booth 1 shows continuous parameter updates:

```csharp
AnimecsController.SetFloat(entityManager, entity, speedHash, 2f);
```

### Transition Patterns

Booth 2 demonstrates state sequencing with parameters:

```csharp
// Idle → Walk
AnimecsController.SetBool(entityManager, entity, walkHash, true);

// Walk → Run  
AnimecsController.SetBool(entityManager, entity, runHash, true);
AnimecsController.SetBool(entityManager, entity, walkHash, false);
```

### Manual LOD Control

Booth 4 forces specific LOD levels:

```csharp
AnimecsController.SetLODLevel(entityManager, entity, AnimecsLODLevel.Medium);
```

### Event Processing

Booth 5 reads triggered events:

```csharp
foreach (var evt in eventBuffer)
{
    if (evt.EventName == "FootStep")
    {
        PlayFootstepAudio(evt.FloatParameter);
    }
}
eventBuffer.Clear();
```

### Input-Driven Animation

Booth 7 shows player input controlling animation:

```csharp
// Read input, set Speed parameter based on movement
float speed = math.length(velocity);
AnimecsController.SetFloat(entityManager, entity, speedHash, speed);

// Jump on input
if (input.JumpPressed)
{
    AnimecsController.SetTrigger(entityManager, entity, jumpHash);
}
```

## Using Sample Code in Your Project

The sample is fully documented and Burst-compiled. Copy any system or pattern you need:

**Want automatic transitions?** See Booth 2's parameter cycling  
**Need event handling?** Copy `AnimecsDemoEventAudioSystem`  
**Building character controller?** Use Booth 7's input system as template  
**Implementing LOD?** See Booth 4's manual override pattern  
**Debugging animations?** Check Booth 8's debug visualization setup

---