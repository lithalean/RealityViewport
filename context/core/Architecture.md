# RealityViewport Architecture

**Purpose**: Technical blueprint and system design for Entity/ECS architecture  
**Version**: 3.0  
**Status**: Production Ready - ~70% Complete  
**Last Updated**: August 2025

## Architectural Overview

### Core Architecture Formula
```
Entity/ECS + Metal Rendering + Adaptive UI + RealityKit = Professional 3D Editor
```

### Critical Architecture Changes (v2.0 → v3.0)
```diff
- Node System (BaseSceneNode hierarchy)
+ Entity/ECS System (Unity DOTS-inspired)

- CPU-based grid rendering
+ GPU Metal pipeline (Sky + Grid)

- Platform-specific views (iPhoneView, MacView)
+ Single adaptive ContentView (WWDC25 standard)

- Node-based selection
+ Entity-based selection with SceneEntity protocol
```

## System Architecture Layers

```
┌─────────────────────────────────────────────┐
│            Application Layer                 │
│         RealityViewportApp.swift            │
├─────────────────────────────────────────────┤
│            UI Layer (SwiftUI)               │
│    ContentView (Adaptive) + Inspector       │
├─────────────────────────────────────────────┤
│          Entity/ECS Layer                   │
│   Entity, CameraEntity, LightEntity, etc.   │
├─────────────────────────────────────────────┤
│           Manager Layer                     │
│  Scene, Selection, Project, Control, DayNight│
├─────────────────────────────────────────────┤
│         Rendering Layer (Hybrid)            │
│    Metal (Sky/Grid) + RealityKit (3D)       │
├─────────────────────────────────────────────┤
│          Platform Layer                     │
│      iOS / macOS / tvOS Abstractions        │
└─────────────────────────────────────────────┘
```

## Core Design Patterns

### Pattern 1: Entity/ECS Hybrid Architecture
```swift
PURPOSE: Unity-like ease with RealityKit performance
IMPLEMENTATION: 
  - Entity wrapper class around RealityKit.Entity
  - SceneEntity protocol for polymorphic behavior
  - Observable properties for SwiftUI binding
BENEFITS:
  - Familiar API for Unity developers
  - Native RealityKit performance
  - Type-safe component access
```

### Pattern 2: Metal + RealityKit Rendering Composition
```swift
PURPOSE: GPU-accelerated backgrounds with 3D content
IMPLEMENTATION:
  - Metal renders sky gradient and grid
  - RealityKit renders 3D entities
  - Transparent layer composition
BENEFITS:
  - 60fps performance
  - Dynamic atmospheric effects
  - Minimal CPU overhead
```

### Pattern 3: ViewportState with RealityKit.Entity
```swift
PURPOSE: Centralized viewport state using native types
IMPLEMENTATION:
  - Uses RealityKit.Entity directly (not custom Entity)
  - PerspectiveCamera for viewport camera
  - Observable for SwiftUI updates
BENEFITS:
  - No type confusion
  - Direct RealityKit integration
  - Clean separation of concerns
```

### Pattern 4: Manager-Driven Architecture
```swift
PURPOSE: Domain-specific responsibility separation
MANAGERS:
  - SceneManager: Entity lifecycle and scene graph
  - SelectionManager: Multi-selection with gizmos
  - ProjectManager: Persistence and file I/O
  - ControlManager: Input handling
  - DayNightManager: Atmospheric lighting
BENEFITS:
  - Single responsibility principle
  - Easy testing and debugging
  - Clear data flow
```

### Pattern 5: Protocol-Based Entity System
```swift
PURPOSE: Flexible entity types with common interface
IMPLEMENTATION:
  protocol SceneEntity {
    var id: UUID { get }
    var position: SIMD3<Float> { get set }
    var realityEntity: RealityKit.Entity { get }
  }
BENEFITS:
  - Polymorphic entity handling
  - Type-safe collections
  - Protocol extensions for shared behavior
```

### Pattern 6: Adaptive UI (WWDC25 Standard)
```swift
PURPOSE: Single view hierarchy for all platforms
IMPLEMENTATION:
  - NavigationSplitView for regular sizes
  - NavigationStack for compact sizes
  - Platform-specific modifiers only when needed
BENEFITS:
  - 100% code reuse
  - Automatic adaptation
  - Future-proof for new devices
```

## Component Architecture

```
RealityViewportApp
├── Managers (EnvironmentObject)
│   ├── SceneManager
│   ├── ProjectManager
│   └── SelectionManager (via SceneManager)
│
├── ContentView (Adaptive)
│   ├── ViewportStack
│   │   ├── MetalSkyView (Layer 0)
│   │   ├── ViewportMetalGrid (Layer 1)
│   │   └── ViewportView (Layer 2)
│   │       ├── RealityView (Entities)
│   │       ├── ViewportToolbar
│   │       └── CameraController
│   │
│   └── Inspector (Adaptive)
│       ├── OutlinerView
│       └── PropertiesView
│
└── Sheets/Modals
    └── ProjectBrowserView
```

## Entity System Architecture

### Entity Hierarchy
```
SceneEntity (Protocol)
    │
    ├── Entity (Base Class)
    │   ├── realityEntity: RealityKit.Entity
    │   ├── position: SIMD3<Float>
    │   ├── rotation: simd_quatf
    │   └── scale: SIMD3<Float>
    │
    ├── CameraEntity : Entity
    │   ├── fov: Float
    │   └── nearPlane/farPlane: Float
    │
    ├── LightEntity : Entity
    │   ├── lightType: LightType
    │   ├── intensity: Float
    │   └── color: SIMD3<Float>
    │
    └── ModelEntity : Entity
        ├── modelURL: URL?
        └── isLoading: Bool
```

### Component Management
```swift
// ECS-style component access
entity.realityEntity.components.set(ModelComponent(...))
entity.realityEntity.components[CollisionComponent.self]
entity.realityEntity.components.remove(InputTargetComponent.self)

// Custom components
EntitySelectionComponent
BillboardComponent
TransformGizmoComponent
```

## State Management Architecture

### State Flow
```
User Input → Manager → @Published Property → SwiftUI View → RealityKit/Metal
```

### Key State Containers
```swift
// Scene State
SceneManager {
    @Published var entities: [any SceneEntity]
    @Published var selectedEntity: (any SceneEntity)?
    var realityEntities: [UUID: RealityKit.Entity]
}

// Viewport State  
ViewportState {
    let rootEntity: RealityKit.Entity  // Note: RealityKit type
    let cameraEntity: PerspectiveCamera
    var lightEntities: [UUID: RealityKit.Entity]
    @Published var needsUpdate: Bool
}

// Project State
ProjectManager {
    @Published var currentProject: ProjectFile?
    @Published var recentProjects: [ProjectFile]
}
```

## Rendering Pipeline Architecture

### Layer Composition
```
1. Metal Sky Renderer
   - Full-screen gradient
   - Day/night cycle
   - GPU shader-based

2. Metal Grid Renderer  
   - Ground plane grid
   - Distance-based fade
   - Colored axes

3. RealityKit Scene
   - 3D entities
   - Gizmos
   - Billboard icons

Composition: Transparent overlay with Color.clear backgrounds
```

### Render Loop
```swift
// 60fps target (16.67ms budget)
Frame {
    1. Update day/night phase (0.1ms)
    2. Render sky gradient (0.5ms)
    3. Render grid (0.3ms)
    4. Update entity transforms (1ms)
    5. Render RealityKit scene (8-12ms)
    6. SwiftUI composition (2ms)
    Total: ~12-15ms (leaving headroom)
}
```

## Critical Type Disambiguation

### Entity Namespace Issues
```swift
// PROBLEM: Two Entity types exist
Entity              // Your custom wrapper class
RealityKit.Entity   // Native RealityKit type

// SOLUTION: Always disambiguate
let customEntity = Entity()                    // Your Entity
let rkEntity = RealityKit.Entity()            // RealityKit's
let wrapped = customEntity.realityEntity      // Returns RealityKit.Entity

// ViewportState uses RealityKit.Entity directly
class ViewportState {
    let rootEntity = RealityKit.Entity()      // NOT custom Entity
    let cameraEntity = PerspectiveCamera()    // RealityKit subclass
}
```

## Platform Abstraction Layer

### Cross-Platform Types
```swift
// PlatformColor.swift
#if os(macOS)
    typealias PlatformColor = NSColor
#else
    typealias PlatformColor = UIColor
#endif

// PlatformImage
#if os(macOS)
    typealias PlatformImage = NSImage
#else
    typealias PlatformImage = UIImage
#endif
```

### Platform-Specific Features
```swift
// Adaptive modifiers
.if(os: .macOS) { view in
    view.navigationBarTitleDisplayMode(.inline)
}

// Platform capabilities
#if os(iOS)
    hapticFeedback(.light)
#endif
```

## Anti-Patterns to Avoid

### ❌ NEVER: Confuse Entity Types
```swift
// WRONG - Type mismatch
let entity = Entity()
viewportState.rootEntity = entity  // Error: expects RealityKit.Entity

// CORRECT - Use proper types
let entity = Entity()
viewportState.rootEntity.addChild(entity.realityEntity)
```

### ❌ NEVER: Direct Node References (OLD SYSTEM)
```swift
// WRONG - Node system removed
sceneManager.nodes.append(node)
selectedNode.nodeType = .camera

// CORRECT - Use Entity system
sceneManager.addEntity(entity)
selectedEntity.entityType = .camera
```

### ❌ NEVER: Platform-Specific Views
```swift
// WRONG - Old approach
#if os(iOS)
    iPhoneView()
#else
    MacView()
#endif

// CORRECT - Adaptive UI
ContentView()  // Adapts automatically
```

### ❌ NEVER: CPU-Based Grid Rendering
```swift
// WRONG - Performance impact
ForEach(gridLines) { line in
    Path { ... }
}

// CORRECT - GPU rendering
ViewportMetalGrid()  // Metal shader
```

### ❌ NEVER: Synchronous Model Loading
```swift
// WRONG - Blocks UI
let model = try ModelEntity.loadModel(contentsOf: url)

// CORRECT - Async loading
Task {
    await model.load(from: url)
}
```

## Architectural Decisions Log

### Decision: Entity/ECS Over Node System
**Rationale**: Better performance, Unity-familiar API, native RealityKit integration  
**Implementation**: Entity wrapper with SceneEntity protocol  
**Result**: Cleaner architecture, better performance, easier component composition

### Decision: Metal Rendering Pipeline
**Rationale**: GPU acceleration for grid and sky, 60fps target  
**Implementation**: Metal shaders with transparent composition  
**Result**: Smooth performance, dynamic atmospherics

### Decision: Single Adaptive View
**Rationale**: WWDC25 best practices, maintainability  
**Implementation**: NavigationSplitView/Stack with size class detection  
**Result**: 100% code reuse, automatic platform adaptation

### Decision: RealityKit.Entity in ViewportState
**Rationale**: Avoid type confusion, direct RealityKit integration  
**Implementation**: Explicit RealityKit.Entity types  
**Result**: Clear type boundaries, no ambiguity

### Decision: Manager Pattern Retention
**Rationale**: Proven architecture, clear responsibilities  
**Implementation**: Enhanced managers for Entity system  
**Result**: Smooth migration, familiar patterns

## Performance Considerations

### Memory Management
```swift
// Shared timer for all entities
private static let sharedUpdateTimer = Timer.publish(...)

// Lazy component access
if entity.hasComponent(ModelComponent.self) { ... }

// Entity pooling (future)
EntityPool.get<ModelEntity>()
```

### Optimization Strategies
- Batch entity updates
- Frustum culling for large scenes
- LOD system (planned)
- Texture atlasing (planned)

## Future Architecture Enhancements

### Planned
- [ ] Entity pooling system
- [ ] Async component loading
- [ ] Plugin architecture
- [ ] Undo/Redo system
- [ ] Multi-window support

### Experimental
- [ ] Compute shaders for particles
- [ ] Entity Component System v2
- [ ] Swift Macros for components
- [ ] Distributed rendering

## Architecture Summary

The v3.0 architecture represents a complete modernization:
- **Entity/ECS** replaces Node system entirely
- **Metal rendering** provides GPU acceleration
- **Adaptive UI** follows WWDC25 standards
- **Type safety** with clear namespace boundaries
- **60fps performance** across all platforms

This architecture provides a solid foundation for professional 3D editing while maintaining Apple platform best practices.