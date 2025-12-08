# LaneGraphManager

Central API for lane graph operations. Provides initialization, spatial queries, lane access, and state management.

## Namespace

```csharp
using LaneGraph;
using Unity.Mathematics; // For float3
```

## Properties

### IsInitialized

```csharp
public static bool IsInitialized { get; }
```

Returns `true` if the lane graph has been initialized for the current scene.

**Example:**
```csharp
if (!LaneGraphManager.IsInitialized)
{
    LaneGraphManager.Initialize();
}
```

### Lanes

```csharp
public static Lane[] Lanes { get; }
```

Direct access to the lane array. Returns `null` if not initialized.

## Initialization

### Initialize()

```csharp
public static void Initialize(bool autoInitializeOnSceneLoad = true)
```

Initializes the lane graph for the active scene.

**Parameters:**
- `autoInitializeOnSceneLoad` - If true, automatically reinitializes on scene load

**Example:**
```csharp
void Start()
{
    LaneGraphManager.Initialize();
}
```

### Initialize(int sceneBuildIndex)

```csharp
public static void Initialize(int sceneBuildIndex, bool autoInitializeOnSceneLoad = true)
```

Initializes for a specific scene by build index.

**Parameters:**
- `sceneBuildIndex` - Scene index from build settings
- `autoInitializeOnSceneLoad` - Auto-reinitialize on scene load

## Spatial Queries

### FindClosestLaneIndex()

```csharp
public static int FindClosestLaneIndex(
    float3 position,
    float maxDistance = float.MaxValue,
    bool considerDirection = false,
    float3 direction = default,
    LaneTags tags = LaneTags.None
)
```

Finds the index of the closest lane to a position.

**Parameters:**
- `position` - World position to search from
- `maxDistance` - Maximum search distance (smaller = faster)
- `considerDirection` - If true, prefers lanes aligned with `direction`
- `direction` - Direction vector for alignment filtering
- `tags` - Filter lanes by tags (None = no filter)

**Returns:** Lane index, or `-1` if no lane found

**Examples:**
```csharp
// Simple closest lane
int lane = LaneGraphManager.FindClosestLaneIndex(transform.position);

// Within 50 units only
int lane = LaneGraphManager.FindClosestLaneIndex(transform.position, 50f);

// Match vehicle's direction
int lane = LaneGraphManager.FindClosestLaneIndex(
    transform.position,
    maxDistance: 100f,
    considerDirection: true,
    direction: transform.forward
);

// Vehicle lanes only
int lane = LaneGraphManager.FindClosestLaneIndex(
    transform.position,
    maxDistance: 50f,
    tags: LaneTags.Vehicle
);
```

### FindClosestLane()

```csharp
public static Lane? FindClosestLane(
    float3 position,
    float maxDistance = float.MaxValue,
    bool considerDirection = false,
    float3 direction = default,
    LaneTags tags = LaneTags.None
)
```

Finds the closest lane and returns the full Lane structure.

**Returns:** `Lane?` (nullable Lane struct), or `null` if no lane found

**Example:**
```csharp
Lane? lane = LaneGraphManager.FindClosestLane(transform.position, 50f);
if (lane.HasValue)
{
    float width = lane.Value.LaneConfiguration.Width;
    var points = lane.Value.LanePoints;
}
```

### FindLanesInRadius()

```csharp
public static int[] FindLanesInRadius(
    float3 position,
    float radius,
    LaneTags tags = LaneTags.None
)
```

Finds all lanes within a specified radius.

**Parameters:**
- `position` - Center of search sphere
- `radius` - Search radius
- `tags` - Optional tag filter

**Returns:** Array of lane indices (empty array if none found)

**Example:**
```csharp
int[] nearbyLanes = LaneGraphManager.FindLanesInRadius(
    transform.position,
    radius: 25f,
    tags: LaneTags.Vehicle
);

foreach (int laneIndex in nearbyLanes)
{
    ProcessLane(laneIndex);
}
```

### GetClosestPointOnLane()

```csharp
public static float3 GetClosestPointOnLane(
    int laneIndex,
    float3 position,
    out float distance,
    out int segmentIndex
)
```

Finds the closest point on a lane to a given position.

**Parameters:**
- `laneIndex` - Index of the lane
- `position` - Position to find closest point to
- `distance` - Output: Distance to closest point
- `segmentIndex` - Output: Index of the lane segment

**Returns:** Closest point on the lane (float3)

**Example:**
```csharp
float3 closestPoint = LaneGraphManager.GetClosestPointOnLane(
    laneIndex,
    transform.position,
    out float distance,
    out int segment
);

if (distance > maxDeviation)
{
    // Vehicle is too far from lane
    transform.position = Vector3.Lerp(
        transform.position,
        closestPoint,
        correctionSpeed * Time.deltaTime
    );
}
```

## Lane Access

### GetLane()

```csharp
public static Lane? GetLane(int laneIndex)
```

Gets the complete Lane structure for a lane index.

**Returns:** `Lane?` or `null` if invalid index

**Example:**
```csharp
Lane? lane = LaneGraphManager.GetLane(laneIndex);
if (lane.HasValue)
{
    var config = lane.Value.LaneConfiguration;
    var points = lane.Value.LanePoints;
    var connections = lane.Value.LaneLinkers;
}
```

### GetLaneWidth()

```csharp
public static float GetLaneWidth(int laneIndex)
```

Gets the width of a lane.

### GetLanePoints()

```csharp
public static float3[] GetLanePoints(int laneIndex)
```

Gets the array of points defining the lane's path. Returns `null` if invalid index.

### GetLaneState()

```csharp
public static LaneState GetLaneState(int laneIndex)
```

Gets the current operational state of a lane.

### GetLaneTags()

```csharp
public static LaneTags GetLaneTags(int laneIndex)
```

Gets the tags assigned to a lane.

### GetLaneDirection()

```csharp
public static LaneDirection GetLaneDirection(int laneIndex)
```

Gets the traffic flow direction of a lane.

### IsIntersectionLane()

```csharp
public static bool IsIntersectionLane(int laneIndex)
```

Returns `true` if the lane is part of an intersection component.

## Lane Connections

### GetAllLaneLinkers()

```csharp
public static LaneLinker[] GetAllLaneLinkers(int laneIndex)
```

Gets all connections from a lane.

**Returns:** Array of LaneLinker structures

**Example:**
```csharp
LaneLinker[] connections = LaneGraphManager.GetAllLaneLinkers(laneIndex);

foreach (var linker in connections)
{
    int targetLane = linker.LaneIndex;
    LaneLinkType type = linker.LinkType;
    Debug.Log($"Connected to lane {targetLane} via {type}");
}
```

### GetLaneLinkers()

```csharp
public static LaneLinker[] GetLaneLinkers(int laneIndex, LaneLinkType linkType)
```

Gets connections filtered by type.

**Example:**
```csharp
// Get only forward connections
LaneLinker[] forwards = LaneGraphManager.GetLaneLinkers(
    laneIndex,
    LaneLinkType.Forward
);

// Get intersection connections
LaneLinker[] junctions = LaneGraphManager.GetLaneLinkers(
    laneIndex,
    LaneLinkType.Intersection
);
```

## State Management

### SetLaneState()

```csharp
public static bool SetLaneState(int laneIndex, LaneState state)
```

Changes the operational state of a lane.

**Parameters:**
- `laneIndex` - Index of the lane to modify
- `state` - New state to apply

**Returns:** `true` if successful, `false` if invalid index

**Example:**
```csharp
// Close a lane (road closure)
LaneGraphManager.SetLaneState(laneIndex, LaneState.Closed);

// Reopen the lane
LaneGraphManager.SetLaneState(laneIndex, LaneState.Open);

// Warning state (yellow light)
LaneGraphManager.SetLaneState(laneIndex, LaneState.AboutToClose);
```

### OnLaneStateChanged

```csharp
public static event Action<int, LaneState> OnLaneStateChanged;
```

Event triggered when any lane's state changes.

**Example:**
```csharp
void Start()
{
    LaneGraphManager.OnLaneStateChanged += HandleStateChange;
}

void OnDestroy()
{
    LaneGraphManager.OnLaneStateChanged -= HandleStateChange;
}

void HandleStateChange(int laneIndex, LaneState newState)
{
    if (newState == LaneState.Closed)
    {
        // Reroute vehicles
        // Update visual indicators
    }
}
```

## Pathfinding

### FindConnectedLanes()

```csharp
public static int[] FindConnectedLanes(float3 startPosition, float3 endPosition)
```

Finds a path between two positions using lane connections.

**Returns:** Array of lane indices forming the path, or `null` if no path exists

**Example:**
```csharp
int[] path = LaneGraphManager.FindConnectedLanes(
    startPosition,
    goalPosition
);

if (path != null && path.Length > 0)
{
    foreach (int laneIndex in path)
    {
        NavigateThroughLane(laneIndex);
    }
}
```

## Events

### OnLaneGraphInitialized

```csharp
public static event Action OnLaneGraphInitialized;
```

Triggered when initialization completes successfully.

**Example:**
```csharp
void Awake()
{
    LaneGraphManager.OnLaneGraphInitialized += OnInitComplete;
}

void OnInitComplete()
{
    Debug.Log("Lane graph ready");
    StartSimulation();
}
```

## Error Handling

Always validate operations:

```csharp
// Check initialization
if (!LaneGraphManager.IsInitialized)
{
    Debug.LogWarning("Lane graph not initialized");
    return;
}

// Validate query results
int lane = LaneGraphManager.FindClosestLaneIndex(position);
if (lane < 0)
{
    Debug.Log("No lane found");
    return;
}

// Safe lane access
var points = LaneGraphManager.GetLanePoints(lane);
if (points == null || points.Length < 2)
{
    Debug.LogWarning("Invalid lane points");
    return;
}
```

## Performance Notes

**Initialization:**
- O(n log n) where n = component count
- Call once per scene, not per frame

**Spatial Queries:**
- O(log n + k·m) via BVH
- Use smallest effective `maxDistance`
- Apply tag filters to reduce candidates

**Lane Access:**
- O(1) direct array access
- Cache results when following lanes

**State Changes:**
- O(1) per lane
- Batch updates when possible

## See Also

- [Spatial Queries](spatial-queries.md) - Detailed query system
- [Lane Types](lane-types.md) - Data structures
- [Runtime Queries Tutorial](../manual/tutorials/runtime-queries.md) - Integration examples
