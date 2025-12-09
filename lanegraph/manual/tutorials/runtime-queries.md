# Runtime Queries

Master the LaneGraphManager API to query lanes, find paths, and integrate with your game systems.

## Initialization

Always initialize before querying:

```csharp
using LaneGraph;

public class MyLaneSystem : MonoBehaviour
{
    void Start()
    {
        // Initialize for current scene
        LaneGraphManager.Initialize();
        
        // Or for specific scene
        // LaneGraphManager.Initialize(sceneBuildIndex);
    }
}
```

## Basic Lane Queries

### Find Closest Lane

```csharp
// Simple closest lane
int laneIndex = LaneGraphManager.FindClosestLaneIndex(
    transform.position
);

if (laneIndex >= 0)
{
    Debug.Log($"Found lane: {laneIndex}");
}
```

### With Maximum Distance

```csharp
// Only within 50 units
int laneIndex = LaneGraphManager.FindClosestLaneIndex(
    transform.position,
    maxDistance: 50f
);
```

### With Direction Filter

```csharp
// Find lane in vehicle's forward direction
float3 position = transform.position;
float3 direction = transform.forward;

int laneIndex = LaneGraphManager.FindClosestLaneIndex(
    position,
    maxDistance: 100f,
    considerDirection: true,
    direction: direction
);
```

### With Tag Filter

```csharp
// Find only vehicle lanes
int laneIndex = LaneGraphManager.FindClosestLaneIndex(
    transform.position,
    maxDistance: 50f,
    considerDirection: false,
    direction: float3.zero,
    tags: LaneTags.Vehicle
);
```

## Getting Lane Information

### Access Lane Data

```csharp
Lane? lane = LaneGraphManager.GetLane(laneIndex);

if (lane.HasValue)
{
    // Get lane properties
    float width = lane.Value.LaneConfiguration.Width;
    LaneDirection dir = lane.Value.LaneConfiguration.Direction;
    LaneTags tags = lane.Value.LaneConfiguration.Tags;
    float speed = lane.Value.LaneConfiguration.Speed;
    LaneState state = lane.Value.CurrentLaneState;
    
    // Get lane geometry
    float3[] points = lane.Value.LanePoints;
    
    // Check connections
    bool isIntersection = lane.Value.IsIntersectionLane;
    LaneLinker[] connections = lane.Value.LaneLinkers;
}
```

### Quick Property Access

```csharp
// Direct property getters
float width = LaneGraphManager.GetLaneWidth(laneIndex);
float3[] points = LaneGraphManager.GetLanePoints(laneIndex);
LaneState state = LaneGraphManager.GetLaneState(laneIndex);
LaneTags tags = LaneGraphManager.GetLaneTags(laneIndex);
LaneDirection dir = LaneGraphManager.GetLaneDirection(laneIndex);
```

## Spatial Queries

### Find Lanes in Radius

```csharp
// Get all lanes within radius
int[] nearbyLanes = LaneGraphManager.FindLanesInRadius(
    position: transform.position,
    radius: 25f
);

foreach(int laneIndex in nearbyLanes)
{
    Debug.Log($"Nearby lane: {laneIndex}");
}
```

### With Tag Filter

```csharp
// Only pedestrian lanes
int[] pedestrianLanes = LaneGraphManager.FindLanesInRadius(
    position: transform.position,
    radius: 25f,
    tags: LaneTags.Pedestrian
);
```

### Get Closest Point on Lane

```csharp
// Find where on the lane is closest
float3 closestPoint = LaneGraphManager.GetClosestPointOnLane(
    laneIndex,
    position: transform.position,
    out float distance,
    out int segmentIndex
);

Debug.Log($"Closest point at distance: {distance}");
Debug.Log($"On segment: {segmentIndex}");
```

## Lane Connections

### Get Connected Lanes

```csharp
// Get all connections
LaneLinker[] connections = LaneGraphManager.GetAllLaneLinkers(laneIndex);

foreach(var linker in connections)
{
    int targetLane = linker.LaneIndex;
    LaneLinkType linkType = linker.LinkType;
    
    Debug.Log($"Connected to lane {targetLane} via {linkType}");
}
```

### Filter by Connection Type

```csharp
// Get only forward connections
LaneLinker[] forwardLinks = LaneGraphManager.GetLaneLinkers(
    laneIndex,
    LaneLinkType.Forward
);

// Get merge connections
LaneLinker[] merges = LaneGraphManager.GetLaneLinkers(
    laneIndex,
    LaneLinkType.Merge
);
```

## Pathfinding

### Find Connected Path

```csharp
// Find lanes connecting two positions
int[] pathLanes = LaneGraphManager.FindConnectedLanes(
    startPosition,
    endPosition
);

if (pathLanes != null && pathLanes.Length > 0)
{
    Debug.Log($"Found path with {pathLanes.Length} lanes");
    
    // Follow the path
    foreach(int laneIndex in pathLanes)
    {
        // Navigate through each lane
    }
}
```

## Practical Integration Examples

### Example 1: Vehicle Lane Following

```csharp
public class VehicleLaneFollower : MonoBehaviour
{
    private int currentLaneIndex = -1;
    private int currentSegment = 0;
    private float laneProgress = 0f;
    
    void Start()
    {
        LaneGraphManager.Initialize();
        FindNearestLane();
    }
    
    void FindNearestLane()
    {
        currentLaneIndex = LaneGraphManager.FindClosestLaneIndex(
            transform.position,
            maxDistance: 50f,
            considerDirection: true,
            direction: transform.forward,
            tags: LaneTags.Vehicle
        );
    }
    
    void Update()
    {
        if (currentLaneIndex < 0) return;
        
        var points = LaneGraphManager.GetLanePoints(currentLaneIndex);
        if (points == null || points.Length < 2) return;
        
        // Find closest point
        var closestPoint = LaneGraphManager.GetClosestPointOnLane(
            currentLaneIndex,
            transform.position,
            out float distance,
            out currentSegment
        );
        
        // Navigate towards next point
        if (currentSegment < points.Length - 1)
        {
            Vector3 targetPoint = points[currentSegment + 1];
            transform.position = Vector3.MoveTowards(
                transform.position,
                targetPoint,
                Time.deltaTime * 10f
            );
        }
        else
        {
            // Reached end, find next lane
            FindNextLane();
        }
    }
    
    void FindNextLane()
    {
        var connections = LaneGraphManager.GetLaneLinkers(
            currentLaneIndex,
            LaneLinkType.Forward
        );
        
        if (connections.Length > 0)
        {
            currentLaneIndex = connections[0].LaneIndex;
            currentSegment = 0;
        }
    }
}
```

### Example 2: AI Navigation

```csharp
public class AINavigator : MonoBehaviour
{
    private List<int> plannedPath;
    private int currentPathIndex = 0;
    
    public void NavigateTo(Vector3 destination)
    {
        // Find path
        plannedPath = new List<int>(
            LaneGraphManager.FindConnectedLanes(
                transform.position,
                destination
            )
        );
        
        currentPathIndex = 0;
        
        if (plannedPath.Count > 0)
        {
            StartCoroutine(FollowPath());
        }
    }
    
    IEnumerator FollowPath()
    {
        while (currentPathIndex < plannedPath.Count)
        {
            int laneIndex = plannedPath[currentPathIndex];
            yield return StartCoroutine(TraverseLane(laneIndex));
            currentPathIndex++;
        }
        
        Debug.Log("Destination reached!");
    }
    
    IEnumerator TraverseLane(int laneIndex)
    {
        var points = LaneGraphManager.GetLanePoints(laneIndex);
        
        foreach(var point in points)
        {
            while (Vector3.Distance(transform.position, point) > 0.5f)
            {
                transform.position = Vector3.MoveTowards(
                    transform.position,
                    point,
                    Time.deltaTime * 5f
                );
                yield return null;
            }
        }
    }
}
```

### Example 3: Traffic Density Monitor

```csharp
public class TrafficDensityMonitor : MonoBehaviour
{
    public float checkRadius = 100f;
    public float updateInterval = 2f;
    
    private Dictionary<int, int> laneVehicleCounts = new Dictionary<int, int>();
    
    void Start()
    {
        LaneGraphManager.Initialize();
        InvokeRepeating(nameof(UpdateTrafficDensity), 0f, updateInterval);
    }
    
    void UpdateTrafficDensity()
    {
        laneVehicleCounts.Clear();
        
        // Find all nearby lanes
        var lanes = LaneGraphManager.FindLanesInRadius(
            transform.position,
            checkRadius
        );
        
        // Count vehicles on each lane
        var vehicles = FindObjectsOfType<Vehicle>();
        
        foreach(var vehicle in vehicles)
        {
            int lane = LaneGraphManager.FindClosestLaneIndex(
                vehicle.transform.position,
                maxDistance: 10f
            );
            
            if (lane >= 0)
            {
                if (!laneVehicleCounts.ContainsKey(lane))
                    laneVehicleCounts[lane] = 0;
                    
                laneVehicleCounts[lane]++;
            }
        }
        
        // Report congested lanes
        foreach(var kvp in laneVehicleCounts)
        {
            if (kvp.Value > 5)
            {
                Debug.Log($"Lane {kvp.Key} is congested with {kvp.Value} vehicles");
            }
        }
    }
}
```

## Performance Tips

### Cache Lane Data

```csharp
// Bad - queries every frame
void Update()
{
    var points = LaneGraphManager.GetLanePoints(laneIndex);
    // Use points...
}

// Good - cache when lane doesn't change
private float3[] cachedPoints;

void OnLaneChanged(int newLane)
{
    cachedPoints = LaneGraphManager.GetLanePoints(newLane);
}

void Update()
{
    // Use cachedPoints...
}
```

### Batch Queries

```csharp
// Query once, use multiple times
void ProcessNearbyLanes()
{
    var nearbyLanes = LaneGraphManager.FindLanesInRadius(
        transform.position, 50f
    );
    
    // Now process all lanes
    foreach(int lane in nearbyLanes)
    {
        ProcessLane(lane);
    }
}
```

### Use Appropriate Max Distance

```csharp
// Too large - wastes performance
FindClosestLaneIndex(pos, maxDistance: 1000f);

// Better - only what's needed
FindClosestLaneIndex(pos, maxDistance: 20f);
```

## Common Patterns

### Pattern: Snap to Lane

```csharp
void SnapToLane()
{
    int lane = LaneGraphManager.FindClosestLaneIndex(transform.position);
    
    var closestPoint = LaneGraphManager.GetClosestPointOnLane(
        lane, transform.position, out _, out _
    );
    
    transform.position = closestPoint;
}
```

### Pattern: Stay in Lane

```csharp
void KeepInLane()
{
    if (currentLaneIndex < 0) return;
    
    var closestPoint = LaneGraphManager.GetClosestPointOnLane(
        currentLaneIndex,
        transform.position,
        out float distance,
        out _
    );
    
    // If too far from lane, pull back
    if (distance > maxOffLaneDistance)
    {
        transform.position = Vector3.Lerp(
            transform.position,
            closestPoint,
            Time.deltaTime * correctionSpeed
        );
    }
}
```

### Pattern: Lane Change

```csharp
bool CanChangeLane(int targetLane)
{
    // Check if lanes are parallel/adjacent
    var currentPoints = LaneGraphManager.GetLanePoints(currentLaneIndex);
    var targetPoints = LaneGraphManager.GetLanePoints(targetLane);
    
    // Check distance between lanes
    float laneDist = Vector3.Distance(currentPoints[0], targetPoints[0]);
    
    return laneDist < laneChangeThreshold;
}
```

## Next Steps

- [Best Practices Guide](../best-practices.md) - Optimization techniques
- [API Reference](../../api/) - Complete API documentation
- [Component Types](../component-types.md) - Understanding components

---

Master these query techniques to build sophisticated lane-based navigation!
