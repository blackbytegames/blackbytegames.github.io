# Lane Transitions

Lane transitions handle merges and splits – where the number of lanes changes. This tutorial covers the Transition Component.

## What are Lane Transitions?

Transitions are used when:
- **Merging**: Multiple lanes combine into fewer lanes
- **Splitting**: One lane divides into multiple lanes

**Common scenarios**: Highway on-ramps, lane reductions, exit ramps

## Creating a Transition

### Step 1: Add Transition Component

1. Right-click in Hierarchy
2. Select **GameObject > Lane > Transition Lane**
3. Position at the transition point

### Step 2: Assign Profile

1. Select the transition
2. Assign a Lane Profile in Inspector
3. Starts with 2 nodes by default

## Configuring Merge Transitions

### Understanding Merges

A merge combines multiple lanes into fewer:
```
[Lane 1] ─┐
[Lane 2] ─┤ → [Merged Lane]
[Lane 3] ─┘
```

### Setting Up a Merge

1. Select transition component
2. In Inspector, find **Lane Transitions** section
3. Click **Enable Merge**
4. Configure:
   - **From Lanes**: Which lanes to merge (indices)
   - **To Lane**: Target merged lane (index)
   - Node points auto-adjust for smooth transition

### Example: Highway On-Ramp

```
Profile: 3 lanes
Node 0 (Start): 3 separate lanes
Node 1 (End): 2 merged lanes

Merge Configuration:
  Lane 0 + Lane 1 → Lane 0 (main highway lanes continue)
  Lane 2 (on-ramp) → Lane 1 (merges in)
```

## Configuring Split Transitions

### Understanding Splits

A split divides lanes into more:
```
              ┌─ [Lane 1]
[Lane] ───────┼─ [Lane 2]
              └─ [Lane 3]
```

### Setting Up a Split

1. Select transition component
2. In **Lane Transitions** section
3. Click **Enable Split**
4. Configure:
   - **From Lane**: Source lane (index)
   - **To Lanes**: Destination lanes (indices)

### Example: Exit Ramp

```
Profile: 2 lanes at start, 3 at end
Node 0 (Start): 2 lanes
Node 1 (End): 3 lanes

Split Configuration:
  Lane 0 → Lane 0, Lane 2 (main splits to main + exit)
  Lane 1 → Lane 1 (continues straight)
```

## Node Configuration

### Positioning Nodes

1. Use Move Tool
2. Position start node where transition begins
3. Position end node where transition completes
4. Distance determines transition smoothness

### Transition Length

Longer transitions are smoother:
- **Short** (< 20 units): Abrupt, visible lane change
- **Medium** (20-50 units): Comfortable for vehicles
- **Long** (50+ units): Very smooth, highway-appropriate

## Multiple Transitions

### Complex Scenarios

You can enable both merge and split simultaneously:

```
Example: Interchange
  Start: 3 lanes
  Middle: Lanes rearrange
  End: 3 lanes (different configuration)
  
  Merge: Lanes 0+1 → Lane 0
  Split: Lane 2 → Lanes 1+2
```

### Configuration Order

1. Enable both Merge and Split
2. Configure merge first
3. Then configure split
4. System handles intermediate lane arrangement

## Practical Examples

### Example 1: Simple On-Ramp

```csharp
// Create transition for 2-lane highway + 1 on-ramp
Profile: "Highway Merge" (3 lanes)

Node 0 position: Where on-ramp meets highway
Node 1 position: 40 units ahead (smooth merge distance)

Merge Configuration:
  - From Lanes: [1, 2]  // On-ramp and right lane
  - To Lane: 1          // Merged right lane
```

### Example 2: Lane Drop

```csharp
// Reduce from 3 lanes to 2 lanes
Profile: Start with 3 lanes, end with 2

Node 0: 3 lanes active
Node 1: 2 lanes active

Merge Configuration:
  - Lanes 1 and 2 merge into Lane 1
  - Lane 0 continues as Lane 0
```

### Example 3: Complex Interchange

```csharp
// Highway split for exit ramp
Start: 3 lanes
End: 3 lanes (but rightmost is exit)

Split Configuration:
  - Lane 2 → Lanes 2, 3
  - Lanes 0, 1 continue straight
```

## Connecting to Other Components

### To Paths

1. Position transition endpoint near path start/end
2. Automatic snapping will connect them
3. Verify lane properties match

### To Intersections

1. Transitions can connect to intersection nodes
2. Useful for complex junction entries/exits
3. Ensure profile compatibility

## Next Steps

- [Traffic Control Tutorial](traffic-control.md) - Control lane availability
- [Runtime Queries Tutorial](runtime-queries.md) - Query transitions at runtime
- [Best Practices Guide](../best-practices.md) - Optimization tips

---

Master transitions to create realistic, smooth lane networks!
