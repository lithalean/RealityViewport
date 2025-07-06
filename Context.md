# RealityViewport — AI Context Document

**Date**: July 6, 2025  
**Status**: Active Development - Phase 2  
**Purpose**: Context engineering for AI systems working with RealityViewport codebase

---

## Project Identity

**RealityViewport** is the scene editor component of Orchard. It is NOT:
- A standalone application
- Called "RealityEditor" (legacy name)
- Part of "Reality Game Engine" (placeholder name)
- Cross-platform beyond Apple ecosystem

**Correct Context**: RealityViewport is a SwiftUI + RealityKit scene editor that runs natively on Apple Silicon, designed as the primary 3D editing interface for Orchard projects.

---

## Technical Architecture Deep Dive

### Core Philosophy
- **Apple Silicon Exclusive**: ARM64 only, no Intel support, no Rosetta
- **Latest APIs Only**: Uses macOS 26+, iOS 26+, Swift 6, RealityKit 5
- **Zero Legacy Code**: No AppKit, minimal UIKit, SwiftUI-first
- **Cross-Platform**: Single codebase for macOS and iOS with platform adaptations

### File Structure & Key Components

```
RealityViewport/
├── App/
│   ├── RealityViewportApp.swift    # Main app entry point with SwiftData
│   └── ContentView.swift           # Platform router (Mac/iPhone/TV)
├── Core/                           # Business logic layer
│   ├── Managers/
│   │   ├── SceneManager.swift      # @Observable scene state
│   │   ├── SelectionManager.swift  # Multi-object selection logic
│   │   └── ControlManager.swift    # Input handling coordination
│   ├── Nodes/                      # Scene graph entities
│   │   ├── SceneNode.swift         # Base protocol for 3D objects
│   │   ├── CameraNode.swift        # Viewport camera representation
│   │   ├── LightNode.swift         # Light sources (directional, point)
│   │   └── ModelNode.swift         # 3D model containers
│   ├── Components/                 # RealityKit components
│   │   ├── EditorComponents.swift  # Selection, gizmo, billboard
│   │   ├── TransformGizmo.swift    # 3D manipulation widgets
│   │   └── BillboardComponent.swift # Camera-facing UI elements
│   └── Systems/
│       └── BillboardSystem.swift   # Update billboard orientations
├── Viewport/                       # RealityKit rendering layer
│   ├── ViewportView.swift          # Main RealityView container
│   ├── CameraController.swift      # Orbit/pan/fly camera logic
│   ├── ViewportState.swift         # Viewport-specific state management
│   ├── ViewportEntityFactory.swift # Entity creation & updates
│   └── ViewportToolbar.swift       # Mode switching UI
├── Inspector/                      # Property editing panels
│   ├── InspectorView.swift         # Main inspector container
│   ├── OutlinerView.swift          # Scene hierarchy display
│   └── PropertiesView.swift        # Object property editor
└── Views/                          # Platform-specific layouts
    ├── Mac/
    │   └── MacView.swift           # HSplitView layout with resizable panels
    └── iPhone/
        └── iPhoneView.swift        # Overlay layout with sliding inspector
```

### Key State Management Patterns

**SceneManager** (`@StateObject`):
- Central source of truth for scene data
- Manages nodes array, selection state, entity updates
- Uses `@Published` for reactive UI updates
- Implements async model loading with proper error handling

**ViewportState** (`@StateObject`):
- Manages viewport-specific state (camera position, interaction mode)
- Handles dual interaction modes: `.environment` vs `.entity`
- Coordinates gizmo updates and billboard orientations
- Throttles updates for 120fps performance

**Interaction Mode System**:
```swift
enum ViewportInteractionMode {
    case environment  // Camera navigation only
    case entity      // Object manipulation only
}
```
- **Environment Mode**: Drag = camera orbit/pan, tap = no effect
- **Entity Mode**: Drag = gizmo interaction, tap = selection cycling
- **Mode Switching**: Manual via toolbar, strict separation enforced

### Platform Adaptations

**MacView**: 
- HSplitView with resizable inspector panel (280pt width)
- Viewport on left, inspector on right
- Toolbar overlay on viewport
- Drag gesture for resizing properties panel

**iPhoneView**:
- Full-screen viewport with overlay inspector
- Sliding inspector from right edge (280pt width)
- Bottom toolbar with material background
- Touch-optimized gesture handling

### RealityKit Integration Patterns

**Entity Factory Pattern**:
- `ViewportEntityFactory.createEntity(for: SceneNode)` creates RealityKit entities
- Maintains entity-to-node mapping via `EntitySelectionComponent`
- Updates entity transforms when node properties change
- Handles light entity special cases with `ViewportState.lightEntities`

**Component System**:
- `EntitySelectionComponent`: Links entities to scene nodes
- `BillboardComponent`: Makes UI elements face camera
- `GizmoAxisComponent`: Handles transform gizmo interactions
- `InputTargetComponent`: Enables entity selection

**Update Cycle**:
1. Scene changes trigger `viewportState.needsUpdate = true`
2. RealityView update block processes pending changes
3. `BillboardSystem.update()` orients UI elements to camera
4. Entity transforms sync with scene node properties

### Performance Considerations

**Update Throttling**:
- Scene manager updates throttled to 16ms (60fps baseline)
- RealityView updates only when `needsUpdate` flag set
- Gesture deltas filtered to prevent micro-movements

**Memory Management**:
- Entities properly removed when nodes deleted
- Entity dictionaries cleared when scene resets
- Weak references where appropriate to prevent cycles

**Apple Silicon Optimizations**:
- Metal 3 rendering pipeline
- Unified memory access patterns
- SIMD math operations for transforms
- Native ARM64 compilation

---

## Current Development Status

### Working Features
- ✅ Cross-platform UI (Mac split view, iPhone overlay)
- ✅ Scene graph with cameras, lights, models
- ✅ Dual interaction modes (environment/entity)
- ✅ Transform gizmos (translate, rotate, scale)
- ✅ USDZ/Reality file import
- ✅ Multi-selection system
- ✅ Orbit/pan/fly camera controls
- ✅ Real-time viewport updates

### Known Issues
- Gizmo hit detection needs refinement
- Entity mode transform sensitivity requires tuning
- iPhone inspector gesture conflicts possible
- Need better visual feedback for mode switching

### Phase 2 Goals (Current)
- Improved multi-selection gestures
- Scene persistence with SwiftData
- Undo/redo system with Swift 6 actors
- Advanced node relationships (parent/child)
- Reality Composer Pro asset pipeline

---

## Development Guidelines for AI

### Code Patterns to Follow
- Use `@Observable` macro for new state objects (Swift 5.9+)
- Prefer `async/await` over completion handlers
- Use SIMD types for all 3D math operations
- Platform-specific code with `#if os(macOS)` / `#if os(iOS)`
- Error handling with proper Swift 6 patterns

### Code Patterns to Avoid
- AppKit or UIKit APIs (use SwiftUI equivalents)
- Legacy Objective-C patterns
- Intel-specific optimizations or x86_64 code
- Blocking operations on main thread
- Force unwrapping without clear safety

### Common Tasks
**Adding New Node Types**: Extend `SceneNode` protocol, update `ViewportEntityFactory`
**Adding UI Components**: Create in `Views/` with platform adaptations
**Modifying Transforms**: Update both scene node and corresponding entity
**Camera Changes**: Go through `CameraController` for proper state management
**Performance Issues**: Check update throttling and entity lifecycle

### Architecture Decisions Context
- **Why Swift 6?**: Actor-based concurrency for thread-safe scene updates
- **Why RealityKit 5?**: Latest rendering features, Apple Silicon optimization
- **Why No Legacy Support?**: Clean codebase, modern patterns, future-proofing
- **Why Dual Interaction Modes?**: Clear separation of concerns, better UX
- **Why Platform-Specific Views?** Native experience on each platform

---

## Integration Context

RealityViewport integrates with Orchard as:
- **Primary 3D Editor**: Scene composition and object placement
- **Asset Preview**: Visual verification of imported content  
- **Scene Setup**: Camera positioning and lighting design
- **Content Pipeline**: Bridge to Reality Composer Pro workflows

**Related Systems** (external to RealityViewport):
- Asset import pipeline
- Game logic editor
- Material editor
- Animation timeline
- Build system

---

## AI Assistant Guidelines

When working with this codebase:
1. **Always consider platform differences** (Mac vs iPhone UI patterns)
2. **Respect the interaction mode system** (environment vs entity)
3. **Use proper Swift 6 concurrency** for any async operations
4. **Follow Apple Human Interface Guidelines** for new UI
5. **Test on both platforms** when making changes
6. **Maintain the "Apple Silicon exclusive" philosophy**
7. **Use the latest API variants** (avoid deprecated patterns)

The goal is a polished, professional-grade scene editor that feels native to Apple platforms and serves as a showcase for modern Apple development techniques.

---

*Updated July 6, 2025 — RealityViewport Context v2.0*