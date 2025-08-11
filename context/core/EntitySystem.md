# Entity/ECS System Architecture

**Module**: ENTITY_SYSTEM.md  
**Version**: 1.0  
**Created**: August 2025  
**Status**: 🆕 NEW - Primary Reference  
**Architecture**: Unity DOTS-inspired ECS Hybrid

## Overview

RealityViewport implements a sophisticated Entity Component System (ECS) hybrid architecture that wraps RealityKit's native ECS with a Unity-like API for ease of use. This system completely replaces the previous Node-based architecture.

## Core Philosophy

```swift
// OLD (Node System) - REMOVED
class SceneNode: ObservableObject { }

// NEW (Entity System) - CURRENT
class Entity: ObservableObject, Identifiable, Hashable {
    public let realityEntity: RealityKit.Entity
}
```

The Entity system provides:
- **Unity-like ease of use** with familiar APIs
- **RealityKit performance** through native ECS
- **Type safety** with Swift protocols
- **Observable state** for SwiftUI integration

## Architecture Layers

```
┌─────────────────────────────────────┐
│         SwiftUI Views               │
├─────────────────────────────────────┤
│      Entity Wrapper Layer           │  ← Your Entity classes
├─────────────────────────────────────┤
│      SceneEntity Protocol           │  ← Common interface
├─────────────────────────────────────┤
│    RealityKit.Entity (Native)       │  ← RealityKit ECS
├─────────────────────────────────────┤
│         Metal Rendering              │  ← GPU pipeline
└─────────────────────────────────────┘
```

## Entity Class Hierarchy

```
SceneEntity (Protocol)
    ├── Entity (Base Class)
    │   ├── CameraEntity
    │   ├── LightEntity
    │   └── ModelEntity
    └── BaseSceneEntity (Alternative Base)
```

## Core Components

### 1. SceneEntity Protocol

```swift
public protocol SceneEntity: AnyObject, ObservableObject, Identifiable {
    var id: UUID { get }
    var name: String { get set }
    var position: SIMD3<Float> { get set }
    var rotation: simd_quatf { get set }
    var scale: SIMD3<Float> { get set }
    var entityType: SceneEntityType { get }
    var realityEntity: RealityKit.Entity { get }
}
```

**Purpose**: Defines the common interface all entities must implement.

### 2. Entity Base Class

Located: `Entities/Entity.swift`

```swift
@MainActor
public class Entity: ObservableObject, Identifiable, Hashable {
    // Core Properties
    public let id = UUID()
    @Published public var name: String
    @Published public var isEnabled: Bool = true
    
    // Transform Properties
    @Published public var position: SIMD3<Float>
    @Published public var rotation: simd_quatf
    @Published public var scale: SIMD3<Float>
    
    // RealityKit Integration
    public private(set) var realityEntity: RealityKit.Entity
    
    // Hierarchy
    public weak var parent: Entity?
    @Published public private(set) var children: [Entity] = []
}
```

**Key Features**:
- Wraps `RealityKit.Entity` with intuitive API
- Observable properties for SwiftUI
- Automatic sync with RealityKit transforms
- Parent-child hierarchy management

### 3. Entity Types

#### CameraEntity
```swift
public class CameraEntity: Entity {
    @Published public var fov: Float = 60.0
    @Published public var nearPlane: Float = 0.1
    @Published public var farPlane: Float = 1000.0
    @Published public var isActive: Bool = false
}
```

#### LightEntity
```swift
public class LightEntity: Entity {
    public enum LightType: String, Codable, CaseIterable {
        case directional, point, spot, area
    }
    
    @Published public var lightType: LightType
    @Published public var intensity: Float = 1000.0
    @Published public var color: SIMD3<Float> = [1, 1, 1]
    @Published public var range: Float = 10.0
}
```

#### ModelEntity
```swift
public class ModelEntity: Entity {
    @Published public var modelURL: URL?
    @Published public var isLoading: Bool = false
    
    public func load(from url: URL) async
}
```

## Component System

### RealityKit Components Used

```swift
// Rendering
ModelComponent          // Mesh + materials
LightComponent         // Lighting
PerspectiveCameraComponent // Camera

// Interaction
InputTargetComponent   // Hit testing
CollisionComponent     // Physics

// Custom Components
EntitySelectionComponent // Selection state
BillboardComponent      // Always face camera
TransformGizmoComponent // Visual manipulation
```

### Adding Components

```swift
// Add component
entity.realityEntity.components.set(ModelComponent(mesh: mesh, materials: materials))

// Query component
if let model = entity.realityEntity.components[ModelComponent.self] {
    // Use model component
}

// Remove component
entity.realityEntity.components.remove(ModelComponent.self)
```

## Entity Lifecycle

### Creation

```swift
// Create empty entity
let entity = Entity(name: "MyEntity")

// Create typed entity
let camera = CameraEntity(name: "MainCamera")
let light = LightEntity(name: "KeyLight", type: .directional)
let model = ModelEntity(modelNamed: "Character")

// Create with factory methods
let box = Entity.box(size: 1.0, material: SimpleMaterial())
let sphere = Entity.sphere(radius: 0.5)
let group = Entity.group(name: "Props", children: [box, sphere])
```

### Registration with Scene

```swift
// Add to scene
sceneManager.addEntity(entity)

// Entity is automatically:
// 1. Added to scene graph
// 2. Registered with selection system
// 3. Synchronized with RealityKit
// 4. Made observable for UI
```

### Transform Operations

```swift
// Direct property access (Unity-like)
entity.position = SIMD3<Float>(0, 1, 0)
entity.rotation = simd_quatf(angle: .pi/4, axis: [0, 1, 0])
entity.scale = SIMD3<Float>(2, 2, 2)

// Helper methods
entity.translate(by: SIMD3<Float>(1, 0, 0))
entity.rotate(by: SIMD3<Float>(0, .pi/2, 0))
entity.lookAt(target: targetPosition)

// World vs Local
let worldPos = entity.worldPosition
let localPos = entity.position
```

### Hierarchy Management

```swift
// Parent-child relationships
parentEntity.addChild(childEntity)
parentEntity.removeChild(childEntity)
entity.removeFromParent()

// Query hierarchy
let allDescendants = entity.getAllDescendants()
let foundChild = entity.findChild(named: "SpecificChild")

// Iterate children
for child in entity.children {
    // Process child
}
```

### Destruction

```swift
// Remove from scene
sceneManager.removeEntity(entity)

// Cleanup happens automatically:
// 1. Removed from parent
// 2. Children are orphaned
// 3. Components cleaned up
// 4. Unregistered from systems
```

## Scene Management Integration

### SceneManager Operations

```swift
class SceneManager: ObservableObject {
    // Entity storage
    @Published private(set) var entities: [any SceneEntity] = []
    
    // Typed accessors
    var cameraEntities: [CameraEntity]
    var lightEntities: [LightEntity]
    var modelEntities: [ModelEntity]
    
    // Operations
    func addEntity(_ entity: any SceneEntity)
    func removeEntity(_ entity: any SceneEntity)
    func updateEntity(_ entity: any SceneEntity)
    
    // RealityKit bridge
    func getRealityEntity(for entity: any SceneEntity) -> RealityKit.Entity?
}
```

### Selection System

```swift
class SelectionManager: ObservableObject {
    @Published private(set) var primarySelection: (any SceneEntity)?
    @Published private(set) var selection: [any SceneEntity] = []
    
    func select(_ entity: any SceneEntity)
    func addToSelection(_ entity: any SceneEntity)
    func isSelected(_ entity: any SceneEntity) -> Bool
}
```

## RealityKit Bridge

### Namespace Disambiguation

```swift
// CRITICAL: Always disambiguate between your Entity and RealityKit's

// Your custom Entity class
let myEntity = Entity(name: "Custom")

// RealityKit's Entity
let rkEntity = RealityKit.Entity()

// Accessing the wrapped RealityKit entity
let realityEntity = myEntity.realityEntity  // Returns RealityKit.Entity
```

### ViewportState Integration

```swift
class ViewportState: ObservableObject {
    // Uses RealityKit.Entity directly
    let rootEntity = RealityKit.Entity()
    let cameraEntity = PerspectiveCamera()  // RealityKit type
    var lightEntities: [UUID: RealityKit.Entity] = [:]
    var currentGizmo: RealityKit.Entity?
}
```

## Performance Considerations

### Optimization Strategies

1. **Shared Timer for Updates**
   ```swift
   // All entities share one timer instead of individual timers
   private static let sharedUpdateTimer = Timer.publish(every: 1.0/60.0, on: .main, in: .common)
   ```

2. **Lazy Component Access**
   ```swift
   // Only access components when needed
   if entity.hasComponent(ModelComponent.self) {
       let model = entity.getComponent(ModelComponent.self)
   }
   ```

3. **Batch Operations**
   ```swift
   // Update multiple entities in one pass
   sceneManager.batchUpdate(entities) { entity in
       entity.position += delta
   }
   ```

## Common Patterns

### Entity Factory Pattern

```swift
extension Entity {
    static func createLight(type: LightEntity.LightType, intensity: Float) -> LightEntity {
        let light = LightEntity(name: "\(type) Light", type: type)
        light.intensity = intensity
        return light
    }
    
    static func createCamera(fov: Float = 60) -> CameraEntity {
        let camera = CameraEntity(name: "Camera")
        camera.fov = fov
        return camera
    }
}
```

### Component Queries

```swift
// Find all entities with specific components
let modelsWithCollision = entities.filter { entity in
    entity.hasComponent(ModelComponent.self) && 
    entity.hasComponent(CollisionComponent.self)
}
```

### Entity Templates

```swift
// Define reusable entity configurations
struct EntityTemplate {
    static func playerCharacter() -> ModelEntity {
        let player = ModelEntity(modelNamed: "Player")
        player.position = SIMD3<Float>(0, 0, 0)
        player.scale = SIMD3<Float>(1, 1, 1)
        // Add components...
        return player
    }
}
```

## Migration from Node System

### Key Differences

| Node System (OLD) | Entity System (NEW) |
|-------------------|---------------------|
| `SceneNode` protocol | `SceneEntity` protocol |
| `nodeType` property | `entityType` property |
| `CameraNode` class | `CameraEntity` class |
| `LightNode` class | `LightEntity` class |
| `ModelNode` class | `ModelEntity` class |
| `nodes` array | `entities` array |
| `selectedNode` | `selectedEntity` |

### Migration Examples

```swift
// OLD Node System
let node = CameraNode()
node.nodeType = .camera
sceneManager.nodes.append(node)

// NEW Entity System
let entity = CameraEntity()
entity.entityType = .camera  // Set automatically
sceneManager.addEntity(entity)
```

## Best Practices

### 1. Always Use Type-Safe Accessors
```swift
// Good
if let camera = entity as? CameraEntity {
    camera.fov = 45
}

// Avoid
entity.setValue(45, forKey: "fov")  // Not type-safe
```

### 2. Leverage Protocol Conformance
```swift
// All entities conform to SceneEntity
func processAnyEntity(_ entity: any SceneEntity) {
    print("Processing: \(entity.name)")
    entity.position = .zero
}
```

### 3. Maintain Clean Hierarchy
```swift
// Organize entities logically
let environmentGroup = Entity.group(name: "Environment")
environmentGroup.addChild(terrain)
environmentGroup.addChild(sky)
environmentGroup.addChild(foliage)
```

### 4. Use Observable Properties
```swift
// SwiftUI automatically updates when properties change
struct EntityView: View {
    @ObservedObject var entity: Entity
    
    var body: some View {
        Text("Position: \(entity.position)")  // Auto-updates
    }
}
```

## Debugging

### Entity Inspector
```swift
extension Entity {
    func debugDescription() -> String {
        """
        Entity: \(name)
        Type: \(entityType)
        Position: \(position)
        Children: \(children.count)
        Components: \(realityEntity.components.count)
        """
    }
}
```

### Hierarchy Visualization
```swift
func printHierarchy(_ entity: Entity, indent: Int = 0) {
    let padding = String(repeating: "  ", count: indent)
    print("\(padding)├─ \(entity.name) [\(entity.entityType)]")
    for child in entity.children {
        printHierarchy(child, indent: indent + 1)
    }
}
```

## Future Enhancements

### Planned Features
- [ ] Entity pooling for performance
- [ ] Async component loading
- [ ] Entity templates/prefabs
- [ ] Component serialization
- [ ] Entity state machines
- [ ] LOD system integration

### Known Limitations
- SpotLight using PointLight as fallback
- No area lights in RealityKit
- Limited component introspection
- No runtime component creation

## Summary

The Entity/ECS system provides a modern, performant, and developer-friendly architecture that:
- ✅ Replaces the entire Node system
- ✅ Wraps RealityKit with intuitive APIs
- ✅ Integrates seamlessly with SwiftUI
- ✅ Maintains 60fps performance
- ✅ Supports all platforms (iOS, macOS, tvOS)

This is the foundation upon which all other systems (rendering, UI, interaction) are built.