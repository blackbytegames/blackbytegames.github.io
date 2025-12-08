# Best Practices

Follow these guidelines to build efficient, maintainable lane networks.

## Design Principles

### Start Simple
Begin with basic layouts and add complexity incrementally:

1. Create core paths first
2. Add intersections at junctions
3. Insert transitions for merges/splits
4. Fine-tune curves and connections last

### Use Meaningful Profiles
Create profiles that represent real concepts:

```
✓ Good Profile Names
- "Highway_4Lane"
- "City_Street_Bidirectional"
- "Pedestrian_Sidewalk"

✗ Poor Profile Names
- "Profile1"
- "Test"
- "NewLaneProfile"
```

### Organize Hierarchically
Structure components in your scene hierarchy:

```
RoadNetwork/
├─ Highways/
│  ├─ Highway_North
│  └─ Highway_South
├─ City_Streets/
│  ├─ Main_Street
│  └─ Cross_Street
└─ Intersections/
   ├─ Intersection_A
   └─ Intersection_B
```

## Performance Guidelines

### Query Optimization

**Cache lane data** when following a lane:
```csharp
// ✗ Bad - queries every frame
void Update()
{
    var points = LaneGraphManager.GetLanePoints(currentLane);
    // Use points...
}

// ✓ Good - cache when lane changes
private float3[] cachedPoints;

void OnLaneChanged(int newLane)
{
    cachedPoints = LaneGraphManager.GetLanePoints(newLane);
}
```

**Use appropriate search distances:**
```csharp
// ✗ Wasteful - searches entire scene
FindClosestLaneIndex(pos, maxDistance: 10000f);

// ✓ Efficient - targeted search
FindClosestLaneIndex(pos, maxDistance: 20f);
```

**Apply tag filters** to reduce candidates:
```csharp
// ✗ Searches all lanes
FindClosestLaneIndex(pos, tags: LaneTags.None);

// ✓ Searches only vehicle lanes
FindClosestLaneIndex(pos, tags: LaneTags.Vehicle);
```

### Component Design

**Keep path node counts reasonable:**
- Simple roads: 5-10 nodes
- Complex curves: 15-25 nodes
- Split long roads into multiple paths

**Limit intersection complexity:**
- 3-way or 4-way intersections: Standard
- 5-6 way intersections: Use carefully
- 7+ way: Consider redesign or multiple intersections

### Initialization

**Initialize once** at scene start:
```csharp
void Start()
{
    if (!LaneGraphManager.IsInitialized)
    {
        LaneGraphManager.Initialize();
    }
}
```

**Never rebuild during play mode.** Build graph in editor, not at runtime.

## Workflow Practices

### Building Process

**Incremental building:**
1. Create a few components
2. Build and test
3. Add more components
4. Build and test again

This catches errors early and makes debugging easier.

**Always build after:**
- Adding/removing components
- Changing lane profiles
- Modifying component properties
- Updating lane connections

### Testing Strategy

**Verify connections visually:**
- Check lane colors match profiles
- Ensure snapping occurred at endpoints
- Verify smooth curves at junctions

**Test with debug markers:**
```csharp
void OnDrawGizmos()
{
    if (!LaneGraphManager.IsInitialized) return;
    
    int lane = LaneGraphManager.FindClosestLaneIndex(transform.position);
    if (lane >= 0)
    {
        var points = LaneGraphManager.GetLanePoints(lane);
        Gizmos.color = Color.yellow;
        for (int i = 0; i < points.Length - 1; i++)
        {
            Gizmos.DrawLine(points[i], points[i + 1]);
        }
    }
}
```

### Debugging Approach

**Enable debug logs in Project Settings:**
- Set verbosity level to `Verbose` during development
- Check console for component creation messages
- Review initialization logs for issues

**Check common failure points:**
1. Scene in build settings?
2. Lane graph built?
3. Initialize() called?
4. Components have valid profiles?

## Component-Specific Tips

### Path Components

**Use appropriate shape types:**
- **Linear** - Straight sections, grid layouts
- **AutoBezier** - Natural roads, most common
- **Bezier** - Precise artistic control, race tracks

**Maintain consistent spacing:**
Place nodes evenly for smoother curves and better performance.

**Connect endpoints properly:**
Position path ends close to intersections/transitions, let snapping handle alignment.

### Intersection Components

**Plan connections before creating:**
Sketch which lanes connect to which before configuring the component.

**Use adaptive tangents initially:**
Let the system calculate curves automatically, then manually adjust only if needed.

**Test all connection paths:**
Verify every connection works by following it visually in Scene view.

### Transition Components

**Appropriate transition length:**
- Highway speeds (80+ units): 40-60 units
- City speeds (50 units): 25-35 units  
- Slow speeds (< 30 units): 15-20 units

**Merge/split gradually:**
Avoid transitions with extreme angle changes.

## Tag Usage

### Define Tags Semantically

Configure tag names in Project Settings > Lane Graph > Lane Tags:

```
✓ Good Tag Names
- "Vehicle"
- "HeavyTruck"
- "Pedestrian"
- "Bicycle"
- "Emergency"

✗ Poor Tag Names
- "Tag1"
- "MyTag"
- "CustomTagA"
```

### Apply Tags Consistently

Use the same tags across related profiles:

```csharp
// All vehicle profiles use same tags
Highway_Profile: Tags = Vehicle | Emergency
City_Street_Profile: Tags = Vehicle
Truck_Route_Profile: Tags = Vehicle | HeavyTruck
```

### Filter Queries by Tags

```csharp
// Pedestrian pathfinding
int lane = LaneGraphManager.FindClosestLaneIndex(
    position,
    maxDistance: 10f,
    tags: LaneTags.Pedestrian
);

// Emergency vehicle routing
int lane = LaneGraphManager.FindClosestLaneIndex(
    position,
    maxDistance: 50f,
    tags: LaneTags.Emergency
);
```

## State Management

### Update States Centrally

Use LaneStateController for traffic signals. For dynamic conditions, centralize state changes:

```csharp
public class RoadManager : MonoBehaviour
{
    public void CloseRoadSection(int[] laneIndices)
    {
        foreach (int lane in laneIndices)
        {
            LaneGraphManager.SetLaneState(lane, LaneState.Closed);
        }
    }
    
    public void OpenRoadSection(int[] laneIndices)
    {
        foreach (int lane in laneIndices)
        {
            LaneGraphManager.SetLaneState(lane, LaneState.Open);
        }
    }
}
```

### Listen for State Changes

Register once, unregister on cleanup:

```csharp
void OnEnable()
{
    LaneGraphManager.OnLaneStateChanged += HandleStateChange;
}

void OnDisable()
{
    LaneGraphManager.OnLaneStateChanged -= HandleStateChange;
}

void HandleStateChange(int laneIndex, LaneState newState)
{
    // React to change
}
```

## Common Mistakes

### ❌ Don't Query Without Checking

```csharp
// ✗ Crashes if not initialized
var points = LaneGraphManager.GetLanePoints(laneIndex);
```

```csharp
// ✓ Safe
if (LaneGraphManager.IsInitialized)
{
    var points = LaneGraphManager.GetLanePoints(laneIndex);
}
```

### ❌ Don't Ignore Return Values

```csharp
// ✗ Assumes lane exists
int lane = LaneGraphManager.FindClosestLaneIndex(pos);
UseTheLane(lane); // May be -1!
```

```csharp
// ✓ Validates result
int lane = LaneGraphManager.FindClosestLaneIndex(pos);
if (lane >= 0)
{
    UseTheLane(lane);
}
```

### ❌ Don't Repeat Expensive Queries

```csharp
// ✗ Queries every frame
void Update()
{
    var lanes = LaneGraphManager.FindLanesInRadius(pos, 100f);
}
```

```csharp
// ✓ Updates only when needed
void OnPositionChanged()
{
    cachedNearbyLanes = LaneGraphManager.FindLanesInRadius(pos, 100f);
}
```

## Performance Targets

**Initialization:**
- Small scenes (< 100 components): < 10ms
- Medium scenes (100-500 components): < 50ms
- Large scenes (500-2000 components): < 200ms

**Queries:**
- Closest lane: < 0.1ms typical
- Radius search (50 units): < 0.5ms typical
- Large radius (200+ units): Consider caching

**State changes:**
- Individual lane: < 0.01ms
- Batch updates (10+ lanes): < 0.1ms

If exceeding these targets, review component counts and query patterns.

## Documentation

### Comment Your Profiles

Use profile names as documentation:

```
// Instead of "Profile1", use:
"Highway_3Lane_100kph"  // Clear purpose
"Pedestrian_Sidewalk_1.5m"  // Includes dimensions
"Emergency_2Way_Priority"  // Indicates special status
```

### Document Lane Indices

For frequently accessed lanes, maintain a reference:

```csharp
public class RaceTrack : MonoBehaviour
{
    // Main racing line
    public int[] mainLanes = { 0, 1, 2, 5, 8, 9 };
    
    // Pit lane sequence  
    public int[] pitLanes = { 12, 13, 14 };
    
    // Shortcut (opens on lap 3)
    public int[] shortcutLanes = { 20, 21 };
}
```

## Summary

**Do:**
- Initialize once at scene start
- Cache frequently accessed data
- Use appropriate query distances
- Build incrementally and test often
- Apply tag filters

**Don't:**
- Query without checking initialization
- Ignore query return values
- Rebuild during play mode
- Use overly large search distances
- Skip validation of results

Following these practices ensures your lane networks remain performant, maintainable, and bug-free.
