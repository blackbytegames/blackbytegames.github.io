# Tutorial: Traffic Control

Learn how to control lane states dynamically using the Lane State Controller for traffic lights and dynamic road conditions.

## Understanding Lane States

Lanes can be in different states:
- **Open**: Normal operation, vehicles can use
- **Closed**: Blocked, no vehicles allowed
- **AboutToClose**: Warning state (yellow light)
- **Yield**: Proceed with caution

## Creating a Lane State Controller

### Step 1: Add Controller

1. Right-click in Hierarchy
2. Select **GameObject > Lane > Lane State Controller**
3. Position near the intersection you want to control

### Step 2: Find Nearby Components

1. Select the controller
2. Set **Detection Radius** (e.g., 25 units)
3. Click **"Find Nearby Components"**
4. Components within radius are automatically added

### Step 3: Create Signal Groups

Signal groups control sets of lanes together:

1. In Inspector, find **Signal Groups** section
2. Click **Add Signal Group**
3. Name it (e.g., "North-South")
4. Add lane indices to control

## Configuring Signal Timing

For each signal group, configure:

- **Green Duration**: How long lanes stay open (seconds)
- **Yellow Duration**: Warning time before closing (seconds)
- **Initial State**: Starting signal state

Example configuration:
```
North-South Group:
  Green: 30 seconds
  Yellow: 3 seconds
  Initial: Green

East-West Group:
  Green: 30 seconds
  Yellow: 3 seconds
  Initial: Red
```

## Finding Lane Indices

To know which lanes to control:

1. Build the lane graph
2. Run the scene
3. Check console logs or use debugging:

```csharp
void Start()
{
    LaneGraphManager.Initialize();
    
    // Find lanes at position
    var lanes = LaneGraphManager.FindLanesInRadius(
        transform.position, 50f
    );
    
    foreach(int laneIndex in lanes)
    {
        Debug.Log($"Lane {laneIndex} found");
    }
}
```

## Runtime Control

### Automatic Cycling

Check **"Begin Play On Start"** in controller to auto-start signal cycling.

### Manual Control

```csharp
// Get reference to controller
LaneStateController controller = GetComponent<LaneStateController>();

// Start cycling
controller.BeginPlay();

// Stop cycling
controller.EndPlay();

// Get current active group
var activeGroup = controller.GetActiveSignalGroup();
```

### Direct Lane Control

Control individual lanes:

```csharp
// Close a specific lane
LaneGraphManager.SetLaneState(laneIndex, LaneState.Closed);

// Open a lane
LaneGraphManager.SetLaneState(laneIndex, LaneState.Open);

// Listen for changes
LaneGraphManager.OnLaneStateChanged += (index, state) =>
{
    Debug.Log($"Lane {index} changed to {state}");
};
```

## Practical Examples

### Example 1: 4-Way Intersection

```csharp
Signal Group 1 - "North-South":
  Lanes: [0, 1, 2, 3]  // North and South directions
  Green: 30s
  Yellow: 3s
  Initial: Green

Signal Group 2 - "East-West":
  Lanes: [4, 5, 6, 7]  // East and West directions
  Green: 30s
  Yellow: 3s
  Initial: Red

Behavior:
  - North-South starts green
  - After 30s, turns yellow for 3s
  - Then red, East-West turns green
  - Cycle repeats
```

### Example 2: Turn Lane Control

```csharp
Signal Group 1 - "Straight Lanes":
  Lanes: [0, 2, 4, 6]
  Green: 40s

Signal Group 2 - "Left Turn Lanes":
  Lanes: [1, 3, 5, 7]
  Green: 15s
  
// Left turns get separate, shorter green phase
```

### Example 3: Dynamic Road Closure

```csharp
public class RoadClosureController : MonoBehaviour
{
    [SerializeField] private int[] lanesToClose;
    
    public void CloseRoad()
    {
        foreach(int laneIndex in lanesToClose)
        {
            LaneGraphManager.SetLaneState(
                laneIndex, 
                LaneState.Closed
            );
        }
    }
    
    public void OpenRoad()
    {
        foreach(int laneIndex in lanesToClose)
        {
            LaneGraphManager.SetLaneState(
                laneIndex, 
                LaneState.Open
            );
        }
    }
}
```

## Visual Debugging

Enable visual debugging in controller:

1. Check **"Show Debug Visuals"**
2. In Play Mode, lanes show their current state:
   - Green: Lane is Open
   - Yellow: Lane is AboutToClose
   - Red: Lane is Closed

## Advanced Techniques

### Coordinated Signals

Create multiple controllers for coordinated timing:

```csharp
// Controller 1 at Intersection A
// Controller 2 at Intersection B
// Both use same timing for green wave effect

public class SignalCoordinator : MonoBehaviour
{
    public LaneStateController[] controllers;
    public float offset; // Delay between controllers
    
    void Start()
    {
        for(int i = 0; i < controllers.Length; i++)
        {
            StartCoroutine(StartWithDelay(
                controllers[i], 
                i * offset
            ));
        }
    }
    
    IEnumerator StartWithDelay(LaneStateController ctrl, float delay)
    {
        yield return new WaitForSeconds(delay);
        ctrl.BeginPlay();
    }
}
```

### Event-Based Control

React to game events:

```csharp
public class EmergencyVehicleDetector : MonoBehaviour
{
    private int[] priorityLanes;
    
    void OnEmergencyVehicle Detected()
    {
        // Clear priority lanes
        foreach(int lane in priorityLanes)
        {
            LaneGraphManager.SetLaneState(
                lane, 
                LaneState.Open
            );
        }
        
        // Close conflicting lanes
        foreach(int lane in conflictingLanes)
        {
            LaneGraphManager.SetLaneState(
                lane, 
                LaneState.Closed
            );
        }
    }
}
```

## Best Practices

1. **Realistic Timing**: Match real-world signal times
2. **Yellow Phase**: Always include warning time
3. **All-Red Phase**: Consider brief all-red for safety
4. **Test Thoroughly**: Verify no conflicting green signals
5. **Handle Initialization**: Ensure proper startup state

## Common Patterns

### Pattern 1: Simple Alternating

```
Group A: Green → Yellow → Red
Group B: Red → Green → Yellow → Red
Repeat
```

### Pattern 2: Protected Left Turns

```
Phase 1: Straight lanes green
Phase 2: Left turn lanes green  
Phase 3: Opposite direction straight
Phase 4: Opposite direction left turns
```

### Pattern 3: Pedestrian Crossing

```
Phase 1: Vehicle lanes green, crosswalk red
Phase 2: All vehicles red, crosswalk green
```

## Troubleshooting

**Issue**: Signals don't cycle
- Check "Begin Play On Start" is enabled
- Verify signal groups have lane indices
- Ensure LaneGraphManager is initialized

**Issue**: Wrong lanes change
- Double-check lane indices
- Use debug visuals to verify
- Build and test incrementally

**Issue**: Timing seems off
- Check green and yellow durations
- Verify no overlapping signals
- Test in slow motion if needed

## Performance Considerations

- State changes are O(1) operations
- Controllers poll on Update() - keep count reasonable
- Visualizations only active in editor with flag enabled
- Multiple controllers work independently

## Next Steps

- [Runtime Queries Tutorial](runtime-queries.md) - Query system integration
- [Best Practices Guide](../best-practices.md) - Optimization techniques
- [API Reference](../../api/lane-state.md) - Detailed API docs

---

Control your lane network dynamically for realistic traffic simulation!
