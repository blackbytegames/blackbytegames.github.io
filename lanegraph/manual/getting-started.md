# Quick Start

This guide walks you through creating your first lane network in under 2 minutes.

## Prerequisites

- Unity 2019.1 or later
- LaneGraph package installed

## Create a Lane Profile

Lane Profiles define your lane structure. Create one in Project Settings:

1. Open **Edit > Project Settings > Lane Graph**
2. Click **Create New Profile**
3. Name it "Basic Road"
4. Click **Add Lane** twice
5. Configure both lanes:
   - Width: `3.5`
   - Direction: `Forward`
   - Tags: Enable `Default`
   - Speed: `50`
6. Click **Save**

Your profile now contains two forward-facing lanes, each 3.5 units wide.

## Add a Path Component

Create your first path:

1. **GameObject > Lane > Path Lane**
2. Select the new PathLane object
3. In Inspector, assign your "Basic Road" profile
4. The lanes appear in Scene view as colored lines

## Shape the Path

Position and curve your path:

1. Select the path component to activate the Move tool
2. Drag the node spheres (blue spheres typically at the start and end of the lane component) to reposition them
3. In Inspector, set **Shape Type** to `AutoBezier`
4. Add more nodes with the ** Add Node** button
5. Drag nodes to create your desired path

## Build the Lane Graph

Before runtime use, build the graph data:

1. **Tools > LaneGraph > Build LaneGraph**
2. Wait for the completion message
3. Verify no errors in Console

> **Important:** Rebuild after any changes to lane components or profiles.

## Query at Runtime

Test your lane network with this script:

```csharp
using UnityEngine;
using LaneGraph;

public class LaneTest : MonoBehaviour
{
    void Start()
    {
        // Initialize the system
        LaneGraphManager.Initialize();
        
        // Find closest lane
        int laneIndex = LaneGraphManager.FindClosestLaneIndex(
            transform.position,
            maxDistance: 50f
        );
        
        if (laneIndex >= 0)
        {
            Debug.Log($"Found lane {laneIndex}");
            
            // Get lane data
            var lane = LaneGraphManager.GetLane(laneIndex);
            if (lane.HasValue)
            {
                var points = lane.Value.LanePoints;
                Debug.Log($"Lane has {points.Length} points");
            }
        }
    }
}
```

1. Create an empty GameObject, add this script
2. Enter Play Mode
3. Check Console for output

## Troubleshooting

**Lanes don't appear in Scene**
- Ensure profile is assigned
- Check profile has lanes added
- Verify component isn't hidden

**"Scene not in build settings" error**
- **File > Build Settings**
- Click **Add Open Scenes**
- Rebuild lane graph

**No lane found at runtime**
- Check object is within 50 units of path
- Verify Initialize() was called
- Ensure lane graph was built

## Next Steps

**Learn more:**
- [Lane Profiles](manual/lane-profiles.md) - Configure complex lane setups
- [Creating Paths](manual/tutorials/creating-paths.md) - Advanced path techniques
- [Building Intersections](manual/tutorials/building-intersections.md) - Connect multiple paths

**Build something:**
- Create a T-intersection connecting three paths
- Design a circular track with multiple lanes
- Add a highway on-ramp using a transition component

## Key Concepts

**Lane Profile** - Reusable template defining lane count, widths, directions, and properties

**Lane Component** - Scene object that generates lanes based on a profile

**Lane Graph** - Runtime data structure built from all components in your scenes

**BVH System** - Spatial acceleration structure enabling fast lane queries

---

You've created your first lane network! Continue with the [tutorials](manual/tutorials/basic-setup.md) to explore advanced features.
