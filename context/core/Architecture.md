# RealityViewport Architecture Context

**Purpose**: Technical blueprint and system design rationale  
**Version**: 2.0  
**Status**: Active Development - ~70% Complete with Mature Architecture  
**Last Updated**: July 2025

## Key Concepts (AI Quick Reference)

### Core Architecture Pattern
```
SwiftUI + RealityKit + ViewportState + Manager Pattern = Professional 3D Editor
```

### Critical Design Elements
1. **Pure SwiftUI** - No AppKit/UIKit dependencies for true cross-platform
2. **ViewportState Pattern** - Centralized 3D viewport state management
3. **Manager-Driven Architecture** - Specialized managers for distinct domains
4. **Node Inheritance System** - BaseSceneNode with specialized implementations
5. **Dual Selection System** - Entity/Node parallel tracking for UI/RealityKit sync

## System Architecture

### Core Design Pattern
The app follows a **Manager-Driven Architecture** where specialized managers handle distinct domains:
- `SceneManager`: Scene graph and entity management
- `ProjectManager`: File I/O and project persistence
- `SelectionManager`: Selection state and multi-selection
- `ControlManager`: Input modes and interaction states

### Component Architecture
```
RealityViewportApp
├── SceneManager (EnvironmentObject)
├── ProjectManager (EnvironmentObject)
└── ContentView
    ├── ViewportView
    │   ├── RealityView (Scene Rendering)
    │   ├── ViewportToolbar
    │   └── CameraController
    ├── InspectorView
    │   ├── OutlinerView
    │   └── PropertiesView
    └── ProjectBrowserView
```

## Technical Architecture

### State Management
- **SwiftUI @StateObject**: App-level managers
- **@EnvironmentObject**: Cross-view manager access
- **@Published**: Reactive UI updates
- **Async/Await**: All file operations

### Component Communication
```swift
// Pattern: Manager → Published Property → View Update
SceneManager.selectedNodes → OutlinerView (auto-updates)
ProjectManager.currentProject → ViewportToolbar (enables save)
```

### Platform Abstraction
```swift
// PlatformColor.swift handles cross-platform colors
#if os(macOS)
    typealias PlatformColor = NSColor
#else
    typealias PlatformColor = UIColor
#endif
```

## Design Patterns

### Pattern 1: ViewportState Centralization
```
PURPOSE: Single source of truth for all viewport rendering state
IMPLEMENTATION: ObservableObject with @Published properties and needsUpdate trigger
BENEFITS: Deterministic rendering, easier debugging, decoupled UI updates
```

### Pattern 2: Node Inheritance Hierarchy
```
PURPOSE: Unified node type handling with specialized behavior
IMPLEMENTATION: BaseSceneNode class with ModelNode/CameraNode/LightNode subclasses
BENEFITS: Shared transform logic, type safety, protocol conformance
```

### Pattern 3: Platform Dialog Abstraction
```
PURPOSE: Cross-platform file operations with native feel
IMPLEMENTATION: SwiftUI FileImporter/FileExporter with platform-specific handling
BENEFITS: Single codebase, native dialogs per platform, security scoping
```

### Pattern 4: Dual Entity/Node Selection
```
PURPOSE: Bridge UI selection needs with RealityKit entity requirements
IMPLEMENTATION: SelectionManager tracks both nodes and entities in parallel
BENEFITS: Clean SwiftUI bindings, proper RealityKit integration
```

### Pattern 5: Billboard Component System
```
PURPOSE: Always-facing UI elements in 3D space
IMPLEMENTATION: ECS pattern with BillboardSystem
BENEFITS: Consistent icon display across camera angles
```

### Pattern 6: Async Model Loading
```
PURPOSE: Non-blocking asset loading with error handling
IMPLEMENTATION: Async/await with security scope handling and fallbacks
BENEFITS: Responsive UI, graceful error recovery
```

## Anti-Patterns to Avoid

### ❌ NEVER: Direct RealityKit Entity Manipulation
```swift
// WRONG - Breaks abstraction
entity.transform.translation = [1, 0, 0]

// CORRECT - Use node system
modelNode.position = SIMD3(1, 0, 0)
```

### ❌ NEVER: Synchronous File Operations
```swift
// WRONG - Blocks UI
let data = try Data(contentsOf: url)

// CORRECT - Async operations
let data = try await projectManager.loadProject()
```

### ❌ NEVER: Platform-Specific UI Code
```swift
// WRONG - Breaks cross-platform
#if os(macOS)
    NSOpenPanel().runModal()
#endif

// CORRECT - Use SwiftUI
.fileImporter(isPresented: $showImporter)
```

### ❌ NEVER: Direct ViewportState Mutation from Multiple Views
```swift
// WRONG - State conflicts
viewportState.cameraPosition = newPosition // From random view

// CORRECT - Through proper channels
cameraController.updateCamera(position: newPosition)
```

### ❌ NEVER: Bypass Security-Scoped Resource Access
```swift
// WRONG - Will fail in sandboxed environment
let modelEntity = try ModelEntity.load(contentsOf: url)

// CORRECT - Use security scope
let accessing = url.startAccessingSecurityScopedResource()
defer { if accessing { url.stopAccessingSecurityScopedResource() } }
let modelEntity = try ModelEntity.load(contentsOf: url)
```

### ❌ NEVER: Synchronous Model Loading in Main Thread
```swift
// WRONG - UI freeze
let model = try ModelEntity.loadModel(named: "heavy.usdz")

// CORRECT - Async loading
Task {
    let model = try await ModelEntity.loadModelAsync(named: "heavy.usdz")
}
```

### ❌ NEVER: Create Entities Without Collision Setup
```swift
// WRONG - No physics/selection
let entity = ModelEntity(mesh: .generateBox(size: 1))

// CORRECT - Proper setup
let entity = ModelEntity(mesh: .generateBox(size: 1))
entity.generateCollisionShapes(recursive: true)
```

## Architectural Decisions Log

### Decision: Pure SwiftUI Approach
**Rationale**: Maximum code reuse across Apple platforms
**Implementation**: All UI in SwiftUI, custom gestures for platform differences
**Result**: 95% shared code between macOS/iOS/tvOS

### Decision: Manager-Based Architecture
**Rationale**: Clear separation of concerns, testability
**Implementation**: Dedicated managers for each domain
**Result**: Easy to add features without touching core logic

### Decision: Node System Over Direct Entities
**Rationale**: Higher-level abstraction for scene manipulation
**Implementation**: SceneNode protocol with concrete implementations
**Result**: Simplified serialization and undo/redo potential