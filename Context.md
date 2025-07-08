# RealityViewport — AI Context Document

**Date**: January 2025  
**Status**: Active Development - Core Features Working  
**Purpose**: Accurate context for AI systems working with RealityViewport codebase

---

## 🎯 Project Identity

**RealityViewport** is a SwiftUI + RealityKit 3D scene editor for the Orchard game engine.

**Key Facts:**
- Part of Orchard (not "RealityEditor" or standalone)
- Cross-platform: macOS, iOS, and tvOS support
- Pure SwiftUI (no AppKit/UIKit dependencies)
- Uses latest RealityKit for rendering
- Apple Silicon optimized but Intel compatible

---

## 🏗️ Current Architecture

### Directory Structure
```
RealityViewport/
├── App/
│   ├── ContentView.swift          # Platform router & file import
│   └── RealityViewportApp.swift   # Main app with SwiftData
├── Core/
│   ├── Components/
│   │   ├── BillboardComponent.swift    # Camera-facing UI
│   │   ├── EditorComponents.swift      # Selection components
│   │   └── TransformGizmo.swift        # 3D manipulation widgets
│   ├── Managers/
│   │   ├── SceneManager.swift          # Central scene state
│   │   ├── SelectionManager.swift      # Multi-selection logic
│   │   └── ControlManager.swift        # Input coordination
│   ├── Nodes/
│   │   ├── SceneNode.swift            # Base protocol
│   │   ├── CameraNode.swift           # Camera representation
│   │   ├── LightNode.swift            # Light sources
│   │   └── ModelNode.swift            # 3D models
│   └── Systems/
│       └── BillboardSystem.swift       # Billboard updates
├── Inspector/
│   ├── InspectorView.swift            # Container view
│   ├── OutlinerView.swift             # Scene hierarchy
│   └── PropertiesView.swift           # Property editing
├── Viewport/
│   ├── ViewportView.swift             # Main RealityKit view
│   ├── ViewportState.swift            # Viewport state
│   ├── CameraController.swift         # Camera controls
│   ├── ViewportEntityFactory.swift    # Entity creation
│   ├── ViewportToolbar.swift          # Mode switching UI
│   └── ViewportGrid.swift             # Grid helper
└── Views/
    ├── Mac/
    │   └── MacView.swift              # macOS layout
    └── iPhone/
        └── iPhoneView.swift           # iOS layout
```

---

## 🔧 Core Implementation Details

### State Management

#### SceneManager (Main State)
```swift
class SceneManager: ObservableObject {
    @Published var nodes: [any SceneNode] = []
    @Published var activeCamera: CameraNode?
    let selectionManager = SelectionManager()
    
    // Entity tracking
    private var nodeToEntity: [UUID: Entity] = [:]
    private var entityToNode: [Entity: any SceneNode] = [:]
}
```
- Manages all scene nodes (cameras, lights, models)
- Integrates with SelectionManager for multi-selection
- Tracks entity-node relationships
- Sets up default Blender-style scene

#### ViewportState (Viewport-Specific)
```swift
class ViewportState: ObservableObject {
    let rootEntity = Entity()
    let cameraEntity = PerspectiveCamera()
    @Published var needsUpdate: Bool = false
    @Published var interactionMode: ViewportInteractionMode = .environment
}
```
- Manages RealityKit entities
- Controls viewport camera
- Handles interaction mode switching
- Triggers viewport updates

### Interaction System

#### Dual Mode Design
```swift
enum ViewportInteractionMode {
    case environment  // Camera navigation only
    case entity      // Object selection/transform
}
```

**Environment Mode**:
- Drag = orbit camera
- Scroll = zoom
- No object interaction

**Entity Mode**:
- Tap = select object
- Drag = transform via gizmo
- Multi-selection support

### Platform Layouts

#### macOS (MacView.swift)
```
[Viewport with overlays] | [Inspector Panel]
        (Main)           |    (280pt wide)
                        |  - Outliner
                        |  - Properties
```
- HSplitView layout
- Viewport on LEFT (opposite of RealitySyntax)
- Inspector on RIGHT with resizable properties

#### iOS (iPhoneView.swift)
```
[Full Screen Viewport]
    [Sliding Inspector →]
[Bottom Toolbar]
```
- Full-screen viewport
- Inspector slides from RIGHT edge
- Material background toolbar at bottom

---

## 💡 Key Implementation Patterns

### Entity-Node Synchronization
1. SceneNode (data model) changes
2. ViewportState.needsUpdate = true
3. RealityView update block runs
4. ViewportEntityFactory creates/updates entities
5. BillboardSystem orients UI elements

### File Import Flow
1. ContentView shows file picker
2. Files copied to temp location (sandbox workaround)
3. SceneManager.importModel() loads async
4. Creates ModelNode with loaded entity
5. Updates viewport automatically

### Camera System
- CameraController manages orbit/pan logic
- Default Blender-style position: [7.48, 5.34, 6.50]
- Looks at origin [0, 0, 0]
- Spherical coordinate system for orbit

---

## 🚧 Current Development State

### ✅ What's Working
- Platform routing (macOS/iOS/tvOS)
- Basic scene with camera and light
- USDZ/Reality file import
- Dual interaction modes
- Transform gizmos (visual only)
- Billboard UI system
- Camera orbit/pan controls
- Scene hierarchy display

### ⚠️ Known Issues
- Gizmo interaction not fully implemented
- Hit testing needs refinement
- Mode switching feedback minimal
- No persistence yet
- No undo/redo system

### 🔄 In Progress
- Gizmo hit detection and dragging
- Multi-selection gestures
- Scene persistence with SwiftData
- Better mode switching UI

---

## 🛠️ Development Guidelines

### Adding Features
1. **New Node Types**: Extend SceneNode protocol, update ViewportEntityFactory
2. **New Components**: Add to Core/Components, register in entity creation
3. **Platform UI**: Add to both MacView and iPhoneView with adaptations
4. **Viewport Features**: Modify ViewportView and ViewportState

### Code Patterns
- Use `@MainActor` for UI-related classes
- `async/await` for file operations
- Platform checks: `#if os(macOS)`
- Entity components for RealityKit features
- Billboard system for camera-facing UI

### Common Tasks
```swift
// Add a new model
let modelNode = ModelNode(name: "MyModel")
sceneManager.addNode(modelNode)

// Change interaction mode
viewportState.interactionMode = .entity

// Update camera
cameraController.setDistance(15.0)
```

---

## 🔍 Debugging Tips

1. **Mode Issues**: Check ViewportToolbar mode button states
2. **Selection Problems**: Verify EntitySelectionComponent setup
3. **Update Issues**: Ensure needsUpdate flag is set
4. **Import Failures**: Check console for file access errors
5. **Platform Differences**: Test on both macOS and iOS

---

## 📝 Integration Notes

### With RealitySyntax
- Opposite layouts (viewport left vs right)
- Shared component patterns
- Similar platform adaptation approach

### With Orchard
- Scene data will integrate with game logic
- Export to game-ready formats
- Asset pipeline connections

---

*This document reflects actual implementation as of January 2025*