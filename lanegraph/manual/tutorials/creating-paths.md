# Creating Paths

Learn advanced techniques for creating and manipulating path components in LaneGraph.

## Path Shapes

### Linear Paths

Linear paths connect nodes with straight lines:

1. Create a Path Lane
2. Ensure **Shape Type** is set to `Linear`
3. Add nodes as needed
4. Each segment will be perfectly straight

**Best for**: City grids, straight highways, warehouse aisles

### AutoBezier Paths

AutoBezier automatically creates smooth curves:

1. Set **Shape Type** to `AutoBezier`
2. Position nodes to define key points
3. The system calculates optimal curves between nodes
4. Adjust node positions to refine the curve

**Best for**: Organic roads, racing tracks, natural-looking paths

### Bezier Paths

Manual Bezier gives you full control:

1. Set **Shape Type** to `Bezier`
2. Each node has in-tangent and out-tangent handles
3. Drag tangent handles to shape curves precisely
4. Use **BezierType** options per node:
   - **Mirrored**: Tangents mirror each other (smooth)
   - **Aligned**: Tangents stay aligned but different lengths
   - **Free**: Complete independence

**Best for**: Precise curve control, custom shapes, artistic paths

## Multi-Lane Paths

### Creating Multiple Lanes

1. In Lane Profile, add multiple lanes
2. Each lane runs parallel to the path
3. Lanes are offset by their widths
4. All lanes follow the same path shape

### Lane Directions

Configure lane flow:

```
Forward-only lanes:
  All lanes set to Direction.Forward
  [→ Lane 1]
  [→ Lane 2]

Bidirectional road:
  Lane 1: Direction.Forward
  Lane 2: Direction.Backward
  [→ Lane 1]
  [← Lane 2]

One-way multi-lane:
  All forward for highways
  [→ Lane 1]
  [→ Lane 2]
  [→ Lane 3]
```

## Node Management

### Adding Nodes

Add nodes in multiple ways:

**Method 1**: Inspector button
- Click "Add Node" to append at the end

**Method 2**: Insert between nodes
- Right-click a node in Scene view (future feature)
- Select "Insert Node After"

**Method 3**: Duplicate paths
- Duplicate an existing path (Ctrl+D)
- Modify nodes on the new path

### Removing Nodes

1. Select the node in Scene view (number indicator)
2. Click "Remove Node" in Inspector
3. **Warning**: Cannot remove if less than 2 nodes remain

### Reordering Nodes

Nodes must stay in sequence along the path. To reorder:

1. Remove nodes you want to move
2. Add them back in desired order
3. Or create a new path with correct order

## Advanced Node Manipulation

### Precise Positioning

For exact coordinates:

1. Select node by clicking in Scene view
2. In Inspector, find "Selected Node Point" section
3. Enter exact X, Y, Z coordinates in Transform fields

### Snap to Grid

Enable grid snapping:

1. In Unity, enable **Grid Snapping** (Tools > Snap Settings)
2. Set your grid size
3. Hold **Ctrl** (Windows) or **Cmd** (Mac) while dragging nodes

### Aligning Nodes

To align nodes on an axis:

1. Select the path
2. Use Unity's **Vertex Snapping** (V key)
3. Hover over the node you want to align
4. Click to snap to another object's position

## Path Reversal

Reverse lane direction:

1. Select the path
2. Check **"Reverse Direction"** in Inspector
3. All lanes flip their flow direction
4. Forward becomes Backward and vice versa

**Use case**: Create return lanes without rebuilding

## Snapping to Other Components

### Automatic Snapping

Paths automatically snap when:

1. Node is within **SnapDistance** of another component's endpoint
2. Visual hint appears (color change)
3. Release to complete snap
4. Creates seamless connection

### Snap Distance

Adjust snap sensitivity in settings:
- **Edit > Project Settings > Lane Graph > Editor Settings**
- Find **SnapDistance** parameter
- Default: 2.0 units

---

Master these path creation techniques to build any lane network you can imagine!
