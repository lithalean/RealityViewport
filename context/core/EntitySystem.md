# Entity/ECS System Architecture

**Module**: ENTITY_SYSTEM.md  
**Version**: 3.0  
**Architecture**: Simplified Entity Wrapper over RealityKit ECS  
**Philosophy**: Start simple, grow with needs  
**Status**: Primitive System Complete - Proven Architecture ✅  
**Last Updated**: December 2024

## Overview

RealityViewport implements a **simplified Entity wrapper system** that provides a Unity-like API over RealityKit's powerful ECS. Phase 1 proved this architecture works perfectly - colored primitives appear immediately, demonstrating the wrapper's effectiveness.

## Core Philosophy

**"Ship the alpha with what works, add complexity when you need it."**

```swift
// Your simplified Entity wrapper (proven in Phase 1)
class Entity {
    var position: SIMD3<Float>  // What you need now
    var realityEntity: RealityKit.Entity  // Escape hatch to full power
}

// Phase 1 proved this pattern works perfectly!
```

The Entity wrapper provides:
- **Simplified API** for common operations
- **Direct access** to RealityKit.Entity when needed
- **Room to grow** as requirements emerge
- **Observable properties** for SwiftUI integration
- **Immediate visibility** with proper scene graph connection ✅

## Phase 1 Achievement: Colored Primitives

### Factory Methods Added
```swift
// Platform-agnostic color support
#if os(macOS)
typealias PlatformColor = NSColor
#else
typealias PlatformColor = UIColor
#endif

// Colored primitive factories - WORKING PERFECTLY
static func box(size: Float = 1.0, color: PlatformColor = .systemBlue, name: String = "Box") -> Entity {
    let mesh = MeshResource.generateBox(size: size)
    var material = SimpleMaterial()
    material.color = .init(tint: color)
    material.roughness = .init(floatLiteral: 0.5)
    material.metallic = .init(floatLiteral: 0.0)
    return model(mesh: mesh, materials: [material], name: name)
}

static func sphere(radius: Float = 0.5, color: PlatformColor = .systemGreen, name: String = "Sphere") -> Entity
static func cylinder(height: Float = 1.0, radius: Float = 0.5, color: PlatformColor = .systemOrange, name: String = "Cylinder") -> Entity
static func plane(width: Float = 2.0, depth: Float = 2.0, color: PlatformColor = .systemGray, name: String = "Plane") -> Entity
static func cone(height: Float = 1.0, radius: Float = 0.5, color: PlatformColor = .systemPurple, name: String = "Cone") -> Entity
```

### The Critical Bridge Pattern (Fixed in Phase 1)

```swift
// CORRECT PATTERN (discovered in Phase 1):
// 1. Entity creates and configures its realityEntity
let primitive = Entity.box(color: .systemBlue)
// primitive.realityEntity already has ModelComponent with materials

// 2. SceneManager uses it directly
public func addEntity(_ entity: any SceneEntity) {
    let realityEntity = entity.realityEntity  // USE EXISTING!
    rootEntity.addChild(realityEntity)
}

// 3. ViewportState connects the scene graphs
public func setSceneManager(_ manager: SceneManager) {
    rootEntity.addChild(manager.rootEntity)  // CRITICAL CONNECTION
}

// WRONG PATTERN (was causing invisible entities):
// Creating duplicate entities through factories
```

## Evolution Path

### Current State (v3.0) - Primitive System Complete
```
What's Built:
├── Basic transforms (position, rotation, scale) ✅
├── Entity types (Camera, Light, Model) ✅
├── Observable properties for SwiftUI ✅
├── Scene hierarchy management ✅
├── Bridge to RealityKit.Entity ✅
├── Colored primitive factories ✅ NEW
├── Platform color compatibility ✅ NEW
└── Proper scene graph connection ✅ NEW

What's NOT Built (intentionally):
├── Complex animation systems
├── Advanced physics wrappers
├── Particle system abstractions
├── Audio component wrappers
└── Network replication
```

## Architecture Layers (Proven in Phase 1)

```
┌─────────────────────────────────────┐
│         SwiftUI Views               │
├─────────────────────────────────────┤
│    Simplified Entity Wrapper        │  ← Creates configured entities
│   (Colored primitives working!)     │
├─────────────────────────────────────┤
│      SceneEntity Protocol           │  ← Polymorphic interface
├─────────────────────────────────────┤
│    RealityKit.Entity (Full ECS)     │  ← Used directly (not duplicated!)
├─────────────────────────────────────┤
│    Scene Graph Connection           │  ← Fixed in Phase 1
│  (ViewportState → SceneManager)     │
├─────────────────────────────────────┤
│         Metal Rendering              │  ← GPU pipeline
└─────────────────────────────────────┘
```

## Entity Wrapper Design (Enhanced)

### The Simplified Wrapper with Primitives

```swift
@MainActor
public class Entity: ObservableObject, Identifiable, Hashable {
    // === CORE (What every entity needs) ===
    public let id = UUID()
    @Published public var name: String
    
    // === TRANSFORMS (Most common operations) ===
    @Published public var position: SIMD3<Float> {
        didSet { 
            realityEntity.position = position
            transformDidChange()  // Notify observers
        }
    }
    @Published public var rotation: simd_quatf {
        didSet { 
            realityEntity.orientation = rotation
            transformDidChange()
        }
    }
    @Published public var scale: SIMD3<Float> {
        didSet { 
            realityEntity.scale = scale
            transformDidChange()
        }
    }
    
    // === THE CRITICAL BRIDGE (Phase 1 Discovery) ===
    // This is properly configured with components!
    public private(set) var realityEntity: RealityKit.Entity
    
    // === ENTITY TYPE ===
    @Published public var entityType: SceneEntityType = .empty
    
    // === HIERARCHY (Basic parent-child) ===
    public weak var parent: Entity?
    @Published public private(set) var children: [Entity] = []
    
    // === MODEL COMPONENT (For primitives) ===
    @Published public var model: ModelComponent? {
        didSet {
            if let model = model {
                realityEntity.components.set(model)
            } else {
                realityEntity.components.remove(ModelComponent.self)
            }
        }
    }
}
```

## Creating Visible Primitives (Phase 1 Success)

### The Working Pattern

```swift
// 1. Create primitive with color and materials
let box = Entity.box(
    size: 1.0,
    color: .systemBlue,
    name: "Box 1"
)

// 2. Position it smartly (eye level, staggered)
let basePosition = SIMD3<Float>(0, 1.5, -3)
let offset = Float(existingCount) * 1.5
box.position = basePosition + SIMD3<Float>(offset, 0, 0)

// 3. Add to scene (uses entity.realityEntity directly)
sceneManager.addEntity(box)

// 4. It's immediately visible! ✅
```

### Why It Works Now

```swift
// Phase 1 discovered the critical path:

// 1. Entity creates a CONFIGURED realityEntity
public static func box(...) -> Entity {
    let mesh = MeshResource.generateBox(...)
    let material = SimpleMaterial(color: ...)
    
    // This creates an Entity with ModelComponent already set
    return model(mesh: mesh, materials: [material], name: name)
}

// 2. SceneManager uses it WITHOUT creating duplicates
public func addEntity(_ entity: any SceneEntity) {
    // DON'T create new entity with ViewportEntityFactory
    // DO use the existing configured one
    let realityEntity = entity.realityEntity  // ✅
    rootEntity.addChild(realityEntity)
}

// 3. ViewportState connects the scene graphs
public func setSceneManager(_ manager: SceneManager) {
    rootEntity.addChild(manager.rootEntity)  // ✅ Makes everything visible
}
```

## Platform Compatibility (Fixed in Phase 1)

### Color Type Handling

```swift
// Platform-agnostic approach
#if os(macOS)
public typealias PlatformColor = NSColor
#else
public typealias PlatformColor = UIColor
#endif

// Used throughout for cross-platform compatibility
var material = SimpleMaterial()
material.color = .init(tint: PlatformColor.systemBlue)
```

### Publishing Changes Fix (iOS)

```swift
// Problem: "Publishing changes from within view updates"
// Solution: Defer primitive creation

private func addPrimitive(_ type: PrimitiveType) {
    Task { @MainActor in  // Defer to next run loop
        let primitive = Entity.box(...)
        sceneManager.addEntity(primitive)
        viewportState.needsUpdate = true
    }
}
```

## Entity Types with Primitives

### ModelEntity - Now Supports Primitives
```swift
public class ModelEntity: Entity {
    @Published public var modelURL: URL?
    @Published public var isLoading: Bool = false
    
    // Can be created as primitive OR loaded from file
    public static func primitive(_ type: PrimitiveType, color: PlatformColor) -> ModelEntity {
        // Uses Entity's primitive factories
        switch type {
        case .box: return Entity.box(color: color) as! ModelEntity
        case .sphere: return Entity.sphere(color: color) as! ModelEntity
        // ...
        }
    }
    
    public func load(from url: URL) async {
        // Load from USDZ file
    }
}
```

## Scene Graph Architecture (Critical Discovery)

### The Two-Graph Problem (Fixed)

```swift
// BEFORE Phase 1 - Two disconnected graphs:
SceneManager.rootEntity     // Has all entities ✓
    └── Entity 1
    └── Entity 2
    
ViewportState.rootEntity    // Rendering, but empty! ✗
    └── Camera
    └── Grid

// AFTER Phase 1 - Connected graphs:
ViewportState.rootEntity
    └── Camera
    └── Grid
    └── SceneManager.rootEntity  // Connected! ✓
            └── Entity 1
            └── Entity 2
```

### The Connection Pattern

```swift
// In ViewportView's RealityView setup:
RealityView { content in
    // Setup viewport basics
    viewportState.rootEntity.addChild(viewportState.cameraEntity)
    
    // CRITICAL: Connect SceneManager
    viewportState.setSceneManager(sceneManager)  // ✅
    
    content.add(viewportState.rootEntity)
}

// In ViewportState:
public func setSceneManager(_ manager: SceneManager) {
    self.sceneManager = manager
    rootEntity.addChild(manager.rootEntity)  // THE FIX
}
```

## Common Patterns (Updated with Phase 1 Lessons)

### The "Direct Usage" Pattern
```swift
// DON'T create duplicate entities
// ✗ ViewportEntityFactory.createRealityEntity(for: entity)

// DO use the entity's configured realityEntity
// ✓ entity.realityEntity
```

### The "Deferred Creation" Pattern
```swift
// Avoid SwiftUI publishing conflicts
Task { @MainActor in
    // Create entities here, not in view update cycle
    let primitive = Entity.box()
    sceneManager.addEntity(primitive)
}
```

### The "Scene Graph Connection" Pattern
```swift
// Always ensure ViewportState knows about SceneManager
viewportState.setSceneManager(sceneManager)

// This connects:
// ViewportState.rootEntity → SceneManager.rootEntity → All entities
```

## Performance Optimizations (Phase 1)

### Shared Timer Optimization
```swift
// Don't create timer per entity
private static let sharedUpdateTimer = Timer.publish(
    every: 1.0/60.0, 
    on: .main, 
    in: .common
).autoconnect().share()

// All entities share one timer
```

### Direct Property Observation (Grid Sync)
```swift
// For real-time sync, observe properties directly
Publishers.CombineLatest3(
    viewportState.$cameraDistance,
    viewportState.$cameraAzimuth,
    viewportState.$cameraElevation
)
.receive(on: RunLoop.main)
.sink { _ in
    updateMatrixBuffer()
    forceRedraw()
}
// Reduced sync latency from 16ms to <1ms
```

## Migration from ViewportEntityFactory

Phase 1 revealed ViewportEntityFactory was creating duplicate entities. Here's the migration:

| Old Pattern (Broken) | New Pattern (Working) | Why |
|---------------------|----------------------|-----|
| Factory creates new entity | Use entity.realityEntity | Entity already configured |
| Two entities per object | One entity per object | No duplication |
| Materials lost | Materials preserved | Using configured entity |
| Complex abstraction | Direct usage | Simpler and works |

## Best Practices (Proven in Phase 1)

### DO: Configure Entities Completely
```swift
// Good - Entity has everything it needs
let box = Entity.box(color: .systemBlue)
// box.realityEntity has ModelComponent with materials
```

### DON'T: Create Duplicate Entities
```swift
// Bad - Creates empty duplicate
let duplicate = RealityKit.Entity()  // ✗

// Good - Use existing configured entity
let existing = entity.realityEntity  // ✓
```

### DO: Connect Scene Graphs
```swift
// Essential for visibility
viewportState.setSceneManager(sceneManager)
```

### DO: Defer State Updates
```swift
// Avoid publishing conflicts
Task { @MainActor in
    // Create and modify entities here
}
```

## Future Growth Areas

### Near Term (Based on Phase 1 Success)
- [x] Primitive creation with colors ✅ DONE
- [x] Scene graph connection ✅ DONE
- [x] Platform compatibility ✅ DONE
- [ ] Primitive type in outliner (in progress)
- [ ] Material editing UI (Phase 3)
- [ ] Shadow support (Phase 2)

### Medium Term (If Needed)
- [ ] Animation basics
- [ ] Physics simulation
- [ ] Particle effects
- [ ] Audio triggers

### Long Term (Maybe Never)
- [ ] Full ECS wrapper parity
- [ ] Custom component system
- [ ] Entity archetypes
- [ ] Parallel systems

## Known Issues (Minor)

### Remaining from Phase 1
- Gizmo interaction needs smoothing (cosmetic)
- Outliner organization for primitives (in progress)
- Occasional iOS frame delay (acceptable)

### Intentional Limitations
- No animation system yet (use realityEntity)
- No physics wrapper yet (use realityEntity)
- No audio abstractions yet (use realityEntity)

These aren't bugs, they're YAGNI in action.

## Debugging Entities

### Verify Entity Configuration
```swift
// Check if primitive is properly configured
print("Entity: \(entity.name)")
print("Has ModelComponent: \(entity.realityEntity.components.has(ModelComponent.self))")
if let model = entity.realityEntity.components[ModelComponent.self] {
    print("Materials: \(model.materials.count)")
}
```

### Verify Scene Graph
```swift
// Check scene hierarchy
print("ViewportState children: \(viewportState.rootEntity.children.count)")
print("SceneManager children: \(sceneManager.rootEntity.children.count)")
print("Connected: \(viewportState.rootEntity.children.contains(sceneManager.rootEntity))")
```

## See Also
- **Implementation.md** - Phase 1 completion details
- **ViewportState.md** - Scene graph connection
- **SceneManager.md** - Entity lifecycle
- **phase1-report-updated.md** - Full journey

## Summary

The Entity system has **proven itself in Phase 1**:
- ✅ **Colored primitives work** - Immediate visibility achieved
- ✅ **Scene graph connected** - Critical architecture fixed
- ✅ **Platform compatible** - iOS/macOS working
- ✅ **Performance maintained** - 60fps with headroom
- ✅ **Architecture validated** - Simple wrapper pattern works

Phase 1 transformed the Entity system from theory to **proven, working reality**. The simplified wrapper approach combined with proper scene graph connection creates a solid foundation for future growth.