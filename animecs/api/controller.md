# AnimecsController

Static API for runtime animation control. All methods Burst-compatible unless noted.

---

## State Control

### Play

```csharp
static void Play(ref EntityCommandBuffer.ParallelWriter ecb, [EntityIndexInQuery] int index, 
    in Entity entity, in FixedString64Bytes stateName, float transitionDuration, 
    AnimecsCurveType curveType = AnimecsCurveType.Smooth, bool overrideExitTime = false, bool resetTime = true)

static void Play(ref EntityCommandBuffer.ParallelWriter ecb, [EntityIndexInQuery] int index, 
    in Entity entity, string stateName, float transitionDuration, 
    AnimecsCurveType curveType = AnimecsCurveType.Smooth, bool overrideExitTime = false, bool resetTime = true)

static void Play(ref EntityCommandBuffer.ParallelWriter ecb, [EntityIndexInQuery] int index, 
    in Entity entity, int stateIndex, float transitionDuration, 
    AnimecsCurveType curveType = AnimecsCurveType.Smooth, bool overrideExitTime = false, bool resetTime = true)
```

Transitions to target state by name or index.

### SetStatePaused

```csharp
static void SetStatePaused(ref AnimecsStateData stateData, bool paused)
```

Pauses or resumes animation time progression.

### SetKeyframeOnlyMode

```csharp
static void SetKeyframeOnlyMode(ref AnimecsStateData stateData, bool keyframeOnly)
```

Disables frame interpolation.

### SetStateTime

```csharp
static void SetStateTime(EntityManager entityManager, Entity entity, float time)
static void SetStateTime(ref AnimecsStateData stateData, float time)
```

Sets current state playback time in seconds.

---

## Parameter Control

### SetFloat

```csharp
static void SetFloat(EntityManager entityManager, Entity entity, int parameterHash, float value)
static void SetFloat(ref BufferLookup<AnimecsFloatParameter> floatLookup, 
    ref AnimecsParameterBlobAsset paramBlob, in Entity entity, int parameterHash, float value)
```

Sets float parameter value.

### GetFloat

```csharp
static float GetFloat(EntityManager entityManager, Entity entity, int parameterHash)
static float GetFloat(ref BufferLookup<AnimecsFloatParameter> floatLookup, 
    ref AnimecsParameterBlobAsset paramBlob, in Entity entity, int parameterHash)
```

Gets float parameter value. Returns 0 if not found.

### SetInt

```csharp
static void SetInt(EntityManager entityManager, Entity entity, int parameterHash, int value)
static void SetInt(ref BufferLookup<AnimecsIntParameter> intLookup, 
    ref AnimecsParameterBlobAsset paramBlob, in Entity entity, int parameterHash, int value)
```

Sets int parameter value.

### GetInt

```csharp
static int GetInt(EntityManager entityManager, Entity entity, int parameterHash)
static int GetInt(ref BufferLookup<AnimecsIntParameter> intLookup, 
    ref AnimecsParameterBlobAsset paramBlob, in Entity entity, int parameterHash)
```

Gets int parameter value. Returns 0 if not found.

### SetBool

```csharp
static void SetBool(EntityManager entityManager, Entity entity, int parameterHash, bool value)
static void SetBool(ref BufferLookup<AnimecsBoolParameter> boolLookup, 
    ref AnimecsParameterBlobAsset paramBlob, in Entity entity, int parameterHash, bool value)
```

Sets bool parameter value.

### GetBool

```csharp
static bool GetBool(EntityManager entityManager, Entity entity, int parameterHash)
static bool GetBool(ref BufferLookup<AnimecsBoolParameter> boolLookup, 
    ref AnimecsParameterBlobAsset paramBlob, in Entity entity, int parameterHash)
```

Gets bool parameter value. Returns false if not found.

### SetTrigger

```csharp
static void SetTrigger(EntityManager entityManager, Entity entity, int parameterHash)
static void SetTrigger(ref BufferLookup<AnimecsTriggerParameter> triggerLookup, 
    ref AnimecsParameterBlobAsset paramBlob, in Entity entity, int parameterHash)
```

Activates trigger parameter. Automatically resets after transition.

### ResetTrigger

```csharp
static void ResetTrigger(EntityManager entityManager, Entity entity, int parameterHash)
static void ResetTrigger(ref BufferLookup<AnimecsTriggerParameter> triggerLookup, 
    ref AnimecsParameterBlobAsset paramBlob, in Entity entity, int parameterHash)
```

Manually resets trigger parameter to false.

**NOTE**: Methods with `EntityManager` parameter are NOT Burst-compatible and must be used on the main thread.

---

## State Queries

### GetCurrentStateIndex

```csharp
static int GetCurrentStateIndex(in AnimecsStateData stateData)
```

Returns current state index.

### GetCurrentStateTime

```csharp
static float GetCurrentStateTime(in AnimecsStateData stateData)
```

Returns current animation time in seconds.

### IsTransitioning

```csharp
static bool IsTransitioning(in AnimecsStateData stateData)
```

Checks if entity is blending between states.

### GetTransitionProgress

```csharp
static float GetTransitionProgress(in AnimecsStateData stateData)
```

Returns blend progress (0-1). Returns 0 if not transitioning.

### FindStateIndex

```csharp
static int FindStateIndex(ref AnimecsStateMachineBlobAsset stateMachine, in FixedString64Bytes stateName)
```

Finds state index by name. Returns -1 if not found.

### GetStateName

```csharp
static FixedString64Bytes GetStateName(ref AnimecsStateMachineBlobAsset stateMachine, int stateIndex)
static bool GetStateName(ref AnimecsStateMachineBlobAsset stateMachine, int stateIndex, out FixedString64Bytes stateName)
```

Gets state name by index. Returns empty string or false if invalid index.

### GetNormalizedStateTime

```csharp
static float GetNormalizedStateTime(in AnimecsStateData stateData, ref AnimecsStateMachineBlobAsset stateMachine)
```

Returns normalized state time (0-1).

---

## LOD Control

### SetLODCamera

```csharp
static void SetLODCamera(EntityManager entityManager, UnityEngine.Camera camera)
```

Sets camera for LOD calculations and frustum culling. Creates or updates `AnimecsCameraManaged` entity. **NOT Burst-compatible.**

### GetLODConfig

```csharp
static AnimecsLODConfig GetLODConfig(EntityManager entityManager)
```

Retrieves global LOD configuration. Returns default if singleton doesn't exist.

### SetLODConfig

```csharp
static void SetLODConfig(EntityManager entityManager, AnimecsLODConfig config)
```

Updates global LOD configuration.

### SetLODLevel

```csharp
static void SetLODLevel(EntityManager entityManager, Entity entity, AnimecsLODLevel level)
static void SetLODLevel(ref ComponentLookup<AnimecsLOD> lodLookup, in Entity entity, 
    AnimecsLODLevel level, byte updateFrequency)
```

Forces specific LOD level. Disables automatic LOD calculation.

**Levels:** `High` (every frame), `Medium` (every 2 frames), `Low` (every 4 frames), `Off` (no updates)

### GetLODLevel

```csharp
static AnimecsLODLevel GetLODLevel(EntityManager entityManager, Entity entity)
static AnimecsLODLevel GetLODLevel(ref ComponentLookup<AnimecsLOD> lodLookup, in Entity entity)
```

Returns current LOD level.

---

## Utility

### HasStateMachineSupport

```csharp
static bool HasStateMachineSupport(EntityManager entityManager, Entity entity)
```

Checks if entity has required Animecs components (`AnimecsTag`, `AnimecsStateData`, `AnimecsStateMachineBlob`).

---