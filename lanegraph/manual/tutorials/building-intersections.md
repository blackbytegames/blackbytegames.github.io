# Building Intersections

Intersections are where multiple paths meet. This tutorial covers creating and configuring intersection components.

## Understanding Intersections

An intersection in LaneGraph:
- Has 3 or more **node points** (one per incoming/outgoing road)
- Each node can have its own lane profile
- Defines **lane connections** between nodes
- Automatically generates curved lanes between connections

## Creating Your First Intersection

### Step 1: Add Intersection Component

1. Right-click in Hierarchy
2. Select **GameObject > Lane > Intersection Lane**
3. Position it where roads should meet

### Step 2: Assign Initial Profile

1. Select the intersection
2. In Inspector, assign a Lane Profile
3. Default creates 3 nodes

### Step 3: Position Node Points

1. Ensure Move Tool  is active
2. Click and drag each node point (numbered spheres)
3. Position them where incoming roads will connect
4. Typically space them evenly for balanced intersections

## Configuring Lane Connections

### Understanding Connection Setup

For each node point, you configure:
- **From Lane**: Which lane on this node
- **To Lane**: Which lane on target node
- **Connection Type**: Turn type (left, right, straight)

### Setting Up Connections

1. Select intersection
2. In Inspector, expand a node (e.g., "Node Point 1")
3. Find **Lane Connections** section
4. Click **Add Connection**
5. Configure:
   - **To Node Index**: Target node (0-based index)
   - **From Lane Index**: Source lane on current node
   - **To Lane Index**: Destination lane on target node

### Example: T-Intersection

```
Node 0 (Bottom - 2 lanes):
  Lane 0 → Node 1, Lane 0 (straight)
  Lane 1 → Node 2, Lane 1 (left turn)

Node 1 (Top - 2 lanes):
  Lane 0 → Node 2, Lane 0 (right turn)
  Lane 1 → Node 0, Lane 1 (straight)

Node 2 (Right - 2 lanes):
  Lane 0 → Node 0, Lane 0 (left turn)
  Lane 1 → Node 1, Lane 1 (straight)
```

## Per-Node Lane Profiles

### Why Different Profiles?

Different roads meeting at an intersection may have:
- Different lane counts
- Different widths
- Different speed limits

### Assigning Per-Node Profiles

1. Select intersection
2. Expand node point in Inspector
3. Find **Lane Profile** dropdown for that node
4. Assign different profile than intersection default
5. Connections update automatically

### Example: Highway to City Street

```
Node 0: Highway Profile (4 lanes, wide)
Node 1: City Street Profile (2 lanes, narrow)
Node 2: City Street Profile (2 lanes, narrow)
```

## Advanced Node Configuration

### Adaptive Tangent

Enable for better automatic curve shaping:

1. Select node in Inspector
2. Check **"Adaptive Tangent"**
3. System auto-calculates optimal curve direction
4. Useful for complex multi-way intersections

### Manual Tangent Control

For precise control:

1. Uncheck **"Adaptive Tangent"**
2. In Scene view, drag tangent handles
3. Adjust in-tangent and out-tangent independently
4. Fine-tune curve steepness and direction

## Common Intersection Types

### Four-Way Intersection (Cross)

```
Setup:
- 4 nodes, evenly spaced 90° apart
- Same profile on all nodes
- Full connection matrix (all turns possible)

Node positions:
  Node 0: North
  Node 1: East  
  Node 2: South
  Node 3: West
```

### T-Intersection

```
Setup:
- 3 nodes forming "T" shape
- Configure left/right turns and straight paths
- Consider turn lanes in profiles

Example connections:
  Bottom to Left: left turn
  Bottom to Right: right turn
  Left to Right: straight
  (and vice versa for each)
```

### Roundabout

```
Setup:
- 4+ nodes in circular arrangement
- Single-direction connections (clockwise or counter-clockwise)
- Entry/exit lanes connect to circle

Strategy:
  1. Create center circle intersection
  2. Create separate path for circle itself
  3. Connect entry/exit points
```

### Y-Intersection (Merge/Split)

```
Setup:
- 3 nodes forming "Y" shape
- Asymmetric positioning
- Can use Transition component instead

When to use intersection vs transition:
  - Intersection: Different directions, traffic control
  - Transition: Same direction, lane count change
```

## Next Steps

- [Lane Transitions Tutorial](lane-transitions.md) - Merges and splits
- [Traffic Control Tutorial](traffic-control.md) - Add signals
- [Component Types Guide](../component-types.md) - Deep dive into components

---

Intersections are powerful tools for creating realistic road networks!
