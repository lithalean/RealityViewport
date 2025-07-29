# RealityViewport Architecture Context

**Purpose**: Technical blueprint and system design rationale  
**Version**: 1.0  
**Status**: Active Development - Core Features Working  
**Last Updated**: July 2025

## Key Concepts (AI Quick Reference)

### Core Architecture Pattern
```
SwiftUI + RealityKit + Pure Declarative = Cross-Platform 3D Editor
```

### Critical Design Elements
1. **Pure SwiftUI** - No AppKit/UIKit dependencies for true cross-platform
2. **Environment Object Pattern** - SceneManager/ProjectManager injected at app level
3. **Node-Based Scene Graph** - SceneNode → ModelNode/CameraNode/LightNode hierarchy
4. **Component System** - BillboardComponent, TransformGizmo for visual helpers

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

### Pattern 1: Node-Based Scene Graph
```
PURPOSE: Unified representation of 3D scene elements
IMPLEMENTATION: Protocol-based nodes with common SceneNode base
BENEFITS: Type-safe scene manipulation, easy serialization
```

### Pattern 2: Billboard Component System
```
PURPOSE: Always-facing UI elements in 3D space
IMPLEMENTATION: ECS pattern with BillboardSystem
BENEFITS: Consistent icon display across camera angles
```

### Pattern 3: Dual Interaction Modes
```
PURPOSE: Toggle between camera and object manipulation
IMPLEMENTATION: ControlManager with mode state
BENEFITS: Clear user intent, reduced input ambiguity
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