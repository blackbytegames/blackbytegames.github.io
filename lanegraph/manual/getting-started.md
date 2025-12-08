# Getting Started with LaneGraph

This guide will walk you through setting up LaneGraph and creating your first lane network in Unity. By the end, you'll have a working lane system that you can query at runtime.

## Prerequisites

- Unity 2021.3 or later
- LaneGraph package installed in your project
- Basic familiarity with Unity Editor

## Step 1: Open Project Settings

Navigate to **Edit > Project Settings > Lane Graph** to open the LaneGraph project settings window.

![Project Settings](../assets/project-settings.png)

This is where you'll manage all your lane profiles.

## Step 2: Create Your First Lane Profile

A Lane Profile defines the structure of your lanes. Let's create a simple two-lane road:

1. Click **"Create New Profile"** button
2. Name it "Two Lane Road"
3. Click **"Add Lane"** twice to create two lanes
4. Configure the lanes:
   - **Lane 1**: Width: 3.5, Direction: Forward, Tags: Vehicle
   - **Lane 2**: Width: 3.5, Direction: Forward, Tags: Vehicle
5. Click **"Save"** to store the profile

Your profile is now ready to use on any lane component!

## Step 3: Create a Path Component

Now let's add a path to your scene:

1. Right-click in the Hierarchy
2. Select **GameObject > Lane > Path Lane**
3. This creates a new GameObject with a PathComponent

## Step 4: Configure the Path

With your path selected:

1. In the Inspector, find the **Lane Profile** dropdown
2. Select your "Two Lane Road" profile
3. The path will update to show two lanes

You'll see the lanes visualized in the Scene view with colored lines representing your configured tags.

## Step 5: Adjust the Path Shape

You can modify the path's shape:

1. Ensure the **Move Tool (W)** is selected
2. Click on the node points (spheres) in the Scene view
3. Drag them to reshape your path
4. For curved paths:
   - Change **Shape Type** to "Bezier" or "AutoBezier"
   - Drag the tangent handles (smaller spheres) to adjust curves

## Step 6: Add More Nodes

To create a longer path:

1. Select your path
2. In the Inspector, click **"Add Node"**
3. Position the new node in the Scene view
4. Repeat as needed to create your desired path

## Step 7: Create an Intersection

Let's connect multiple paths:

1. Right-click in Hierarchy > **GameObject > Lane > Intersection Lane**
2. Assign your lane profile to the intersection
3. Position the intersection where you want paths to meet
4. In the Inspector, configure **Lane Connections** to define which lanes connect to which

## Step 8: Build the Lane Graph

Before you can use the lane network at runtime, you must build it:

1. Navigate to **Tools > LaneGraph > Build LaneGraph**
2. Wait for the build process to complete
3. A success message will appear when done

> **Important**: You must rebuild the lane graph whenever you make changes to your lane components!

## Step 9: Initialize at Runtime

Create a simple script to initialize and use the lane graph:

```csharp
using UnityEngine;
using LaneGraph;
using Unity.Mathematics;

public class LaneGraphExample : MonoBehaviour
{
    void Start()
    {
        // Initialize the lane graph for the current scene
        LaneGraphManager.Initialize();
        
        // Find the closest lane to this position
        float3 position = transform.position;
        int closestLaneIndex = LaneGraphManager.FindClosestLaneIndex(position);
        
        if (closestLaneIndex >= 0)
        {
            Debug.Log($"Closest lane found: {closestLaneIndex}");
            
            // Get lane information
            var lane = LaneGraphManager.GetLane(closestLaneIndex);
            if (lane.HasValue)
            {
                Debug.Log($"Lane width: {lane.Value.LaneConfiguration.Width}");
                Debug.Log($"Lane points: {lane.Value.LanePoints.Length}");
            }
        }
    }
}
```

## Step 10: Test Your Network

1. Attach your script to a GameObject in the scene
2. Enter Play Mode
3. Check the Console for debug messages
4. Experiment with querying different positions

## Common First-Time Issues

### Lanes Not Appearing in Scene View
- Ensure your lane profile has lanes added
- Check that the profile is assigned to the component
- Verify the component isn't hidden or on a hidden layer

### "Scene not in build settings" Error
- Go to **File > Build Settings**
- Click **"Add Open Scenes"** to add your current scene
- Rebuild the lane graph

### Runtime Initialization Fails
- Make sure you built the lane graph (Tools > LaneGraph > Build)
- Verify the scene is in build settings
- Check the Console for specific error messages

### Lanes Don't Connect at Intersections
- Use the snap feature by positioning components close together
- Verify lane connections are configured in the intersection
- Ensure matching lane profiles at connection points

## Next Steps

Now that you have a basic understanding, explore:

- [Creating Paths Tutorial](tutorials/creating-paths.md) - Advanced path creation techniques
- [Building Intersections Tutorial](tutorials/building-intersections.md) - Complex junction setups
- [Lane Profiles Guide](lane-profiles.md) - Deep dive into profile configuration
- [Runtime Queries Tutorial](tutorials/runtime-queries.md) - Advanced query techniques

## Quick Reference

### Essential Keyboard Shortcuts
- **W**: Move tool (required for editing lane nodes)
- **F**: Frame selected component
- **Ctrl+D**: Duplicate component

### Essential Menu Items
- **GameObject > Lane**: Create lane components
- **Edit > Project Settings > Lane Graph**: Manage profiles
- **Tools > LaneGraph > Build**: Build lane graph data

### Key Inspector Features
- **Lane Profile Dropdown**: Assign profiles
- **Shape Type**: Linear, AutoBezier, Bezier
- **Add/Remove Node**: Manage path points
- **Lane Connections**: Configure intersection routing

---

Congratulations! You've created your first lane network. Continue to the [tutorials](tutorials/basic-setup.md) for more advanced techniques.
