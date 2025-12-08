# LaneGraphManager API

`LaneGraphManager` is the central API for all runtime lane graph operations.

## Namespace

```csharp
using LaneGraph;
```

## Initialization

### Initialize()
Initializes the lane graph for the current scene.

```csharp
public static void Initialize()
```

**Example:**
```csharp
void Start()
{
    LaneGraphManager.Initialize();
}
```

### Initialize(int sceneBuildIndex)
Initializes the lane graph for a specific scene.

```csharp
public static void Initialize(int sceneBuildIndex)
```

**Parameters:**
- `sceneBuildIndex`: Index of scene in build settings

## Spatial Queries

### FindClosestLaneIndex()
Finds the closest lane to a position.

```csharp
public static int FindClosestLaneIndex(
    float3 position,
    float maxDistance = float.MaxValue,
    bool considerDirection = false,
    float3 direction = default,
    LaneTags tags = LaneTags.None
)
```

**Parameters:**
- `position`: World position to search from
- `maxDistance`: Maximum search distance
- `considerDirection`: Filter by direction
- `direction`: Direction vector for filtering
- `tags`: Lane tag filter

**Returns:** Lane index, or -1 if not found

**Example:**
```csharp
int lane = LaneGraphManager.FindClosestLaneIndex(
    transform.position,
    maxDistance: 50f,
    considerDirection: true,
    direction: transform.forward,
    tags: LaneTags.Vehicle
);
```

### FindClosestLane()
Returns the closest lane structure.

```csharp
public static Lane? FindClosestLane(
    float3 position,
    float maxDistance = float.MaxValue,
    bool considerDirection = false,
    float3 direction = default,
    LaneTags tags = LaneTags.None
)
```

**Returns:** Lane structure, or null if not found

### FindLanesInRadius()
Finds all lanes within a radius.

```csharp
public static int[] FindLanesInRadius(
    float3 position,
    float radius,
    LaneTags tags = LaneTags.None
)
```

**Returns:** Array of lane indices

**Example:**
```csharp
int[] nearbyLanes = LaneGraphManager.FindLanesInRadius(
    transform.position,
    25f,
    LaneTags.Vehicle
);
```

### GetClosestPointOnLane()
Gets the closest point on a lane to a position.

```csharp
public static float3 GetClosestPointOnLane(
    int laneIndex,
    float3 position,
    out float distance,
    out int segmentIndex
)
```

**Parameters:**
- `laneIndex`: Index of the lane
- `position`: Position to find closest point to
- `distance`: Output distance to closest point
- `segmentIndex`: Output segment index

**Returns:** Closest point on lane

## Lane Access

### GetLane()
Gets complete lane data.

```csharp
public static Lane? GetLane(int laneIndex)
```

**Returns:** Lane structure, or null if invalid index

### GetLaneWidth()
```csharp
public static float GetLaneWidth(int laneIndex)
```

### GetLanePoints()
```csharp
public static float3[] GetLanePoints(int laneIndex)
```

### GetLaneState()
```csharp
public static LaneState GetLaneState(int laneIndex)
```

### GetLaneTags()
```csharp
public static LaneTags GetLaneTags(int laneIndex)
```

### GetLaneDirection()
```csharp
public static LaneDirection GetLaneDirection(int laneIndex)
```

## Lane Connections

### GetAllLaneLinkers()
Gets all connections for a lane.

```csharp
public static LaneLinker[] GetAllLaneLinkers(int laneIndex)
```

### GetLaneLinkers()
Gets filtered connections.

```csharp
public static LaneLinker[] GetLaneLinkers(
    int laneIndex,
    LaneLinkType linkType
)
```

**Example:**
```csharp
// Get forward connections only
var forwards = LaneGraphManager.GetLaneLinkers(
    laneIndex,
    LaneLinkType.Forward
);
```

## State Management

### SetLaneState()
Changes a lane's operational state.

```csharp
public static bool SetLaneState(
    int laneIndex,
    LaneState state
)
```

**Returns:** True if successful

**Example:**
```csharp
LaneGraphManager.SetLaneState(laneIndex, LaneState.Closed);
```

### OnLaneStateChanged
Event triggered when lane state changes.

```csharp
public static event Action<int, LaneState> OnLaneStateChanged;
```

**Example:**
```csharp
void Start()
{
    LaneGraphManager.OnLaneStateChanged += HandleStateChange;
}

void HandleStateChange(int laneIndex, LaneState newState)
{
    Debug.Log($"Lane {laneIndex} changed to {newState}");
}
```

## Pathfinding

### FindConnectedLanes()
Finds path between two positions.

```csharp
public static int[] FindConnectedLanes(
    float3 startPosition,
    float3 endPosition
)
```

**Returns:** Array of lane indices forming path, or null if no path found

## Properties

### IsInitialized
```csharp
public static bool IsInitialized { get; }
```

Check if system is initialized before queries.

### Lanes
```csharp
public static Lane[] Lanes { get; }
```

Direct access to lane array (read-only).

## Component Access

### GetLaneComponent()
```csharp
public static LaneComponentData? GetLaneComponent(int laneIndex)
```

Gets component data for a lane.

### GetComponentIdFromLane()
```csharp
public static int GetComponentIdFromLane(int laneIndex)
```

## Best Practices

1. **Always Initialize**: Call `Initialize()` before queries
```csharp
if (!LaneGraphManager.IsInitialized)
    LaneGraphManager.Initialize();
```

2. **Cache Results**: Don't query every frame
```csharp
// Cache lane points
private float3[] cachedPoints;
void Start()
{
    cachedPoints = LaneGraphManager.GetLanePoints(laneIndex);
}
```

3. **Use Appropriate maxDistance**: Smaller is faster
```csharp
// Good
FindClosestLaneIndex(pos, maxDistance: 20f);

// Wasteful
FindClosestLaneIndex(pos, maxDistance: 1000f);
```

4. **Check Return Values**:
```csharp
int lane = FindClosestLaneIndex(pos);
if (lane >= 0)
{
    // Use lane
}
```

## See Also

- [Lane Types](lane-types.md)
- [Spatial Queries](spatial-queries.md)
- [Runtime Queries Tutorial](../manual/tutorials/runtime-queries.md)
