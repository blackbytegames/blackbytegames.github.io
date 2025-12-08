# Tutorial: Basic Setup

In this tutorial, you'll learn how to set up a simple lane network from scratch. We'll create a basic two-lane road with a curve.

## Prerequisites

- LaneGraph installed in your Unity project
- Empty scene ready

## Step 1: Create a Lane Profile

First, we need to define what our lanes will look like.

1. Go to **Edit > Project Settings > Lane Graph**
2. Click **"Create New Profile"**
3. Name it "Basic Road"
4. Click **"Add Lane"** two times to create two lanes
5. Configure each lane:
   - **Lane 1**:
     - Width: `3.5`
     - Direction: `Forward`
     - Tags: `Vehicle`
     - Speed: `50`
   - **Lane 2**:
     - Width: `3.5`
     - Direction: `Forward`
     - Tags: `Vehicle`
     - Speed: `50`
6. Click **"Save"**

## Step 2: Create a Path Component

1. In your scene Hierarchy, right-click
2. Select **GameObject > Lane > Path Lane**
3. A new GameObject named "PathLane" appears

## Step 3: Assign the Profile

1. Select your PathLane GameObject
2. In the Inspector, find the **Lane Profile** dropdown
3. Select "Basic Road"
4. Your lanes will now appear in the Scene view!

## Step 4: Position the Path

1. Use the **Move Tool (W)**
2. Position your path where you want it in the scene
3. The initial path has two nodes at the start and end

## Step 5: Add Nodes for a Curve

Let's make our path curved:

1. With PathLane selected, click **"Add Node"** in the Inspector
2. This adds a new node at the end of the path
3. Click **"Add Node"** again to add another one

Now you have 4 nodes total.

## Step 6: Shape the Curve

1. Make sure **Move Tool (W)** is active
2. In the Inspector, change **Shape Type** to `AutoBezier`
3. In the Scene view, click and drag the middle nodes to create a curve
4. The system automatically creates smooth curves between nodes

## Step 7: Visualize Your Lanes

In the Scene view, you should see:
- Two parallel lines (your lanes) following the curve
- Blue spheres at node positions
- Colored lines based on your lane tags
- A boundary box around the entire path

## Step 8: Fine-Tune the Curve (Optional)

For more control:

1. Change **Shape Type** to `Bezier`
2. Small tangent handles appear at each node
3. Drag these handles to adjust the curve shape
4. The in-tangent and out-tangent control curve steepness

## Step 9: Test Different Profiles

Experiment with different configurations:

1. Create a new profile with different settings:
   - Different widths
   - More/fewer lanes
   - Different directions
2. Assign it to your path
3. See how it changes the visualization

## Step 10: Build the Lane Graph

Before testing at runtime:

1. Go to **Tools > LaneGraph > Build LaneGraph**
2. Wait for the build to complete
3. Check the Console for any warnings or errors

## Step 11: Create a Test Script

Create a new C# script named `LaneTest.cs`:

```csharp
using UnityEngine;
using LaneGraph;
using Unity.Mathematics;

public class LaneTest : MonoBehaviour
{
    [SerializeField] private Transform testTarget;
    
    void Start()
    {
        // Initialize the lane graph
        LaneGraphManager.Initialize();
        Debug.Log("Lane Graph Initialized!");
        
        // Test finding closest lane
        if (testTarget != null)
        {
            int closestLane = LaneGraphManager.FindClosestLaneIndex(
                testTarget.position
            );
            
            if (closestLane >= 0)
            {
                Debug.Log($"Closest lane to target: {closestLane}");
                
                // Get lane details
                var lane = LaneGraphManager.GetLane(closestLane);
                if (lane.HasValue)
                {
                    Debug.Log($"Lane width: {lane.Value.LaneConfiguration.Width}m");
                    Debug.Log($"Number of points: {lane.Value.LanePoints.Length}");
                }
            }
            else
            {
                Debug.Log("No lane found nearby");
            }
        }
    }
}
```

## Step 12: Test at Runtime

1. Create an empty GameObject named "LaneTest"
2. Attach the `LaneTest` script to it
3. Create a sphere or cube as a visual target
4. Assign the target to the script's Test Target field
5. Position the target near your lane
6. Enter Play Mode
7. Check the Console for output

## Expected Results

You should see:
- "Lane Graph Initialized!" message
- Information about the closest lane found
- Lane width and point count

## Troubleshooting

**Issue**: Lanes don't appear in Scene view
- **Solution**: Make sure the Lane Profile is assigned and has lanes configured

**Issue**: "Scene not in build settings" error when building
- **Solution**: Go to File > Build Settings, click "Add Open Scenes"

**Issue**: No lane found at runtime
- **Solution**: Ensure target is within reasonable distance of the path (< 100 units)

**Issue**: Build fails with errors
- **Solution**: Check Console for specific errors, ensure all components have valid profiles

## What You Learned

In this tutorial, you:
- Created a lane profile
- Added a path component to your scene
- Shaped the path using nodes
- Built the lane graph data
- Queried lanes at runtime

## Next Steps

Now that you understand the basics, try:
- [Creating Paths Tutorial](creating-paths.md) - Advanced path techniques
- [Building Intersections Tutorial](building-intersections.md) - Connect multiple paths
- [Lane Profiles Guide](../lane-profiles.md) - Deep dive into profile options

## Challenge Exercise

Try creating:
1. A highway on-ramp (hint: use multiple paths)
2. A T-intersection
3. A circular track with multiple lanes

---

Great job completing the basic setup! You're ready for more advanced features.
