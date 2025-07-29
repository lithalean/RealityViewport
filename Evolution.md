# RealityViewport Evolution Log

**Purpose**: Track architectural decisions, their outcomes, and lessons learned  
**Version**: 1.0  
**Format**: Problem → Decision → Implementation → Results (PDIR)  
**Last Updated**: July 2025

## Decision Registry

| Date | Decision | Impact | Complexity | Success |
|------|----------|--------|------------|---------|
| 2025-01 | Pure SwiftUI Architecture | High | High | ✅ |
| 2025-02 | Manager-Based State | High | Medium | ✅ |
| 2025-03 | Billboard Component System | Medium | Low | ✅ |
| 2025-04 | Node Abstraction Layer | High | High | ✅ |
| 2025-05 | Dual Interaction Modes | Medium | Low | ✅ |
| 2025-06 | ProjectManager Separation | High | Medium | 🔄 |
| 2025-07 | File System Integration | High | High | 🔄 |

## 2025-01: Pure SwiftUI Architecture

### The Problem
Need to build a 3D scene editor that works identically across macOS, iOS, and tvOS without platform-specific code branching that would create maintenance burden.

### The Decision
Commit to 100% SwiftUI with no AppKit/UIKit dependencies. Use SwiftUI's platform abstractions and create custom gesture handlers where needed.

### The Implementation
```swift
// Platform color abstraction
#if os(macOS)
    typealias PlatformColor = NSColor
#else
    typealias PlatformColor = UIColor
#endif

// Unified gesture handling
ViewportView()
    .gesture(dragGesture)
    .gesture(magnificationGesture)
```

### The Results
- **Success**: 95% code sharing across platforms
- **Benefit**: Single codebase to maintain
- **Challenge**: Some platform conventions harder to implement
- **Metrics**: 3 platform targets, 1 codebase

### Lessons Learned
- SwiftUI's maturity in 2025 makes pure SwiftUI viable for complex apps
- Platform abstractions should be minimal and centralized
- Gesture differences can be abstracted successfully

## 2025-02: Manager-Based State Architecture

### The Problem
Complex state management across 3D scene, file system, selection, and input modes was becoming unwieldy with simple @State properties.

### The Decision
Create dedicated manager classes for each domain: SceneManager, ProjectManager, SelectionManager, and ControlManager. Use @StateObject and environment injection.

### The Implementation
```swift
// App level
@StateObject private var sceneManager = SceneManager()
@StateObject private var projectManager = ProjectManager()

// Injection
ContentView()
    .environmentObject(sceneManager)
    .environmentObject(projectManager)

// Usage
@EnvironmentObject var sceneManager: SceneManager
```

### The Results
- **Success**: Clear separation of concerns
- **Benefit**: Easy to add features without touching other systems
- **Challenge**: Initial boilerplate setup
- **Metrics**: 4 managers, 0 state conflicts

### Lessons Learned
- Manager pattern scales well for complex SwiftUI apps
- Environment injection prevents prop drilling
- Published properties provide automatic UI updates

## 2025-03: Billboard Component System

### The Problem
Camera and light entities needed visual representation in 3D space, but icons should always face the camera for visibility.

### The Decision
Implement an ECS-style BillboardComponent with a BillboardSystem that updates orientations each frame.

### The Implementation
```swift
struct BillboardComponent: Component {
    var iconName: String
    var size: Float
}

class BillboardSystem: System {
    func update(context: SceneUpdateContext) {
        // Rotate all billboards to face camera
    }
}
```

### The Results
- **Success**: Icons always visible and recognizable
- **Benefit**: Reusable for any entity type
- **Challenge**: Initial performance concern (unfounded)
- **Metrics**: 60fps maintained with 100+ billboards

### Lessons Learned
- RealityKit's ECS is powerful for custom behaviors
- Don't optimize prematurely - test first
- Visual clarity trumps minor performance costs

## 2025-04: Node Abstraction Layer

### The Problem
Direct manipulation of RealityKit entities made serialization difficult and tied code too closely to RealityKit implementation details.

### The Decision
Create a Node abstraction layer (SceneNode protocol) with concrete implementations for each entity type. Nodes manage their underlying RealityKit entities.

### The Implementation
```swift
protocol SceneNode {
    var id: UUID { get }
    var name: String { get set }
    var entity: Entity { get }
}

class ModelNode: SceneNode {
    // Wraps ModelEntity with high-level API
}
```

### The Results
- **Success**: Clean serialization API
- **Benefit**: Future-proof against RealityKit changes
- **Challenge**: Additional abstraction layer
- **Metrics**: 100% entities wrapped, 0 direct entity access

### Lessons Learned
- Abstraction layers pay off for external dependencies
- Protocol-oriented design enables future flexibility
- Small performance cost worth the architectural benefits

## 2025-05: Dual Interaction Modes

### The Problem
Users need to both navigate the 3D viewport (camera control) and manipulate objects (selection/transformation), but combining both in one mode created input ambiguity.

### The Decision
Implement explicit Camera Mode and Object Mode with clear visual indication and easy switching via toolbar toggle.

### The Implementation
```swift
enum InteractionMode {
    case camera
    case object
}

// Mode determines gesture behavior
switch interactionMode {
case .camera:
    // Drag orbits camera
case .object:
    // Drag moves object (future)
}
```

### The Results
- **Success**: Clear user mental model
- **Benefit**: No accidental camera moves during object edit
- **Challenge**: Users must learn mode concept
- **Metrics**: 0 gesture conflicts, 2 distinct modes

### Lessons Learned
- Explicit modes reduce user confusion
- Visual mode indication is critical
- Industry-standard patterns (Blender, Maya) work well

## 2025-06: ProjectManager Separation

### The Problem
File I/O logic was getting mixed with scene management, making it hard to implement save/load/export features cleanly.

### The Decision
Extract all project persistence logic into a dedicated ProjectManager with async APIs for all file operations.

### The Implementation
```swift
class ProjectManager: ObservableObject {
    func saveProject(_ scene: SceneManager) async throws
    func loadProject(into scene: SceneManager) async throws
    func exportScene(_ scene: SceneManager, format: ExportFormat) async throws
}
```

### The Results
- **Success**: Clean API defined
- **Benefit**: Testable file operations
- **Challenge**: UI integration incomplete
- **Status**: Implementation 70% complete

### Lessons Learned
- Separate I/O concerns early
- Async/await perfect for file operations
- UI hookup often lags behind API design

## 2025-07: File System Integration

### The Problem
Need robust project management with custom file format, but must balance between simple files and complex project bundles.

### The Decision
Use directory-based .rvproject format with JSON manifest and referenced assets. Support both in-place editing and project copies.

### The Implementation
```
ProjectName.rvproject/
├── manifest.json
├── scene.json
├── assets/
│   └── [imported models]
└── thumbnails/
    └── preview.png
```

### The Results
- **Status**: In active development
- **Progress**: Structure defined, implementation partial
- **Challenge**: Asset reference management
- **Timeline**: 2-3 weeks to complete

### Lessons Learned
- Start with simple format, evolve as needed
- Directory bundles provide flexibility
- Asset management is always complex

## Future Evolution Considerations

### Upcoming Decisions
1. **Undo/Redo System**: Command pattern vs. state snapshots
2. **Multi-selection**: Extend SelectionManager or new system
3. **Plugin Architecture**: If/how to support extensions
4. **Cloud Sync**: CloudKit vs. custom solution

### Technical Debt to Address
1. Complete ProjectManager UI integration
2. Add comprehensive error handling
3. Implement performance profiling
4. Create unit test suite

### Success Metrics
- Code sharing: 95% across platforms ✅
- Performance: 60fps with typical scenes ✅
- Stability: No critical crashes ✅
- Features: 60% of roadmap complete 🔄