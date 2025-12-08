# Changelog

All notable changes to LaneGraph will be documented in this file.

## [1.5] - 2025-12-09

### Changed
- Converted LaneGraph to a Unity Package Manager (UPM) package. The asset now lives under `Packages/` instead of `Assets/`.

### Fixed
- Reversing a lane’s direction now correctly marks the lane component as not built, updating the boundary gizmo color to indicate that a rebuild is required.
- Lane snapping now selects the nearest compatible lane sequence when multiple matches are found, instead of snapping to the first match returned.

## [1.4] - 2025-07-28
### New Features
- **LOD system for gizmo drawing**  
  Added distance-based and view-based culling for editor gizmos, providing up to 85–95% reduction in gizmo rendering overhead for large scenes.
- **Depth-aware gizmo rendering (ZTest)**  
  Added a ZTest option to lane gizmo drawing so lane boundaries correctly appear behind occluding scene geometry, improving visual clarity when working in dense environments.

### Performance Improvements
- **Optimized gizmo rendering**  
  Replaced multiple individual `Handles.DrawAAPolyLine` calls with single batched calls using more efficient memory allocation, significantly reducing draw-call overhead during scene editing.

### User Experience
- **Redesigned LaneStateController inspector**  
  Complete overhaul of the `LaneStateController` inspector using a more efficient and faster `InspectorGUI` approach, improving both performance and usability when configuring lane states.

### Removals and Cleanup
- **Removed LaneSelectionWindow**  
  Deprecated in favor of the new integrated selection approach.  
  If you are upgrading from a previous version, you can safely delete the `LaneSelectionWindow` script, as it is no longer used.

### Configuration
- **New project settings**  
  Added configurable distance thresholds for gizmo culling (`Max Gizmo Draw Distance`, `Medium Gizmos Detail Distance`) under:  
  `Edit -> Project Settings -> Lane Graph -> Editor Settings` tab.

## [1.3] - 2025-07-25


### Improvements
- Added descriptive tooltips to all key inspector fields across LaneGraph components to make configuration clearer and reduce setup errors.
- Changed Lane Graph project settings class to `public` so it can be accessed and extended more easily by user code and tooling.

### API
- Added `SceneBuildIndex` API for identifying and working with lane graphs per scene build index.
- Added `IsIntersectionLane` API to quickly check whether a lane belongs to an intersection.

### Fixed
- Fixed a null reference exception that occurred when attempting to build the lane graph in a scene with no lanes.

## [1.2] - 2025-04-27

### Performance Improvements
- Audited the project using the Auditor tool and made several changes to reduce overhead, including removing LINQ usage, along with other micro-optimizations.

### Fixed
- Fixed an `Invalid target point index` error in the `IntersectionComponentEditor` when working with intersection target points.

### Examples and Code Quality
- Disabled Read/Write on all objects in the `Examples` folder to avoid unnecessary settings in example content.
- Improved the initialization flow of LaneGraph in example scripts and refactored the code for clearer and more robust setup.

## [1.1] - 2025-04-25

### Fixed
- Fixed an index-out-of-bounds error in the `IntersectionComponent` lane connection inspector when restoring foldout states for lane connections.

## [1.0] - 2025-04-04

First Release

---

For support, contact [sridogames@gmail.com](mailto:sridogames@gmail.com)
