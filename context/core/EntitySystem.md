# Entity/ECS System Architecture

**Module**: ENTITY_SYSTEM.md  
**Version**: 2.0  
**Architecture**: Simplified Entity Wrapper over RealityKit ECS  
**Philosophy**: Start simple, grow with needs  
**Status**: Foundation Complete - Growing System  
**Last Updated**: August 2025

## Overview

RealityViewport implements a **simplified Entity wrapper system** that provides a Unity-like API over RealityKit's powerful ECS. This is not an attempt to replicate Apple's entire Entity system on day one. Instead, it's a pragmatic wrapper that starts with the basics and grows as your game needs evolve.

## Core Philosophy

**"Ship the alpha with what works, add complexity when you need it."**

```swift
// Your simplified Entity wrapper (today)
class Entity {
    var position: SIMD3<Float>  // What you need now
    var realityEntity: RealityKit.Entity  // Escape hatch to full power
}

// Not this (trying to build everything day 1)
class Entity {
    // 500 properties and methods you don't need yet...
}
```

The Entity wrapper provides:
- **Simplified API** for common operations
- **Direct access** to RealityKit.Entity when needed
- **Room to grow** as requirements emerge
- **Observable properties** for SwiftUI integration

## Evolution Path

### Current State (v2.0) - Alpha Foundation
```
What's Built:
├── Basic transforms (position, rotation, scale)
├── Entity types (Camera, Light, Model)
├── Observable properties for SwiftUI
├── Scene hierarchy management
└── Bridge to RealityKit.Entity

What's NOT Built (intentionally):
├── Complex animation systems
├── Advanced physics wrappers
├── Particle system abstractions
├── Audio component wrappers
└── Network replication
```

### Growth Strategy
```
Phase 1 (Current): Basic 3D editor functionality ✅
Phase 2 (As Needed): Add animation when you need moving objects
Phase 3 (As Needed): Add physics when you need collisions
Phase 4 (As Needed): Add particles when you need effects
Phase ∞ (Maybe Never): Full parity with RealityKit.Entity
```

## Architecture Layers

```
┌─────────────────────────────────────┐
│         SwiftUI Views               │
├─────────────────────────────────────┤
│    Simplified Entity Wrapper        │  ← Your growing API
│      (What you need today)          │
├─────────────────────────────────────┤
│      SceneEntity Protocol           │  ← Polymorphic interface
├─────────────────────────────────────┤
│    RealityKit.Entity (Full ECS)     │  ← Always accessible
├─────────────────────────────────────┤
│         Metal Rendering              │  ← GPU pipeline
└─────────────────────────────────────┘
```

## Entity Wrapper Design

### The Simplified Wrapper Approach

```swift
@MainActor
public class Entity: ObservableObject, Identifiable, Hashable {
    // === CORE (What every entity needs) ===
    public let id = UUID()
    @Published public var name: String
    
    // === TRANSFORMS (Most common operations) ===
    @Published public var position: SIMD3<Float> {
        didSet { realityEntity.position = position }
    }
    @Published public var rotation: simd_quatf {
        didSet { realityEntity.orientation = rotation }
    }
    @Published public var scale: SIMD3<Float> {
        didSet { realityEntity.scale = scale }
    }
    
    // === THE ESCAPE HATCH ===
    // When you need something not wrapped yet, use this!
    public let realityEntity: RealityKit.Entity
    
    // === HIERARCHY (Basic parent-child) ===
    public weak var parent: Entity?
    @Published public private(set) var children: [Entity] = []
    
    init(name: String = "Entity") {
        self.name = name
        self.realityEntity = RealityKit.Entity()
        self.position = .zero
        self.rotation = simd_quatf()
        self.scale = .one
    }
}
```

**Design Principles:**
1. **Wrap only what you use** - Don't wrap features you don't need
2. **Keep the escape hatch** - Always allow direct RealityKit access
3. **Observable by default** - SwiftUI integration from day one
4. **Grow on demand** - Add features when you actually need them

## Entity Types (Current Set)

### CameraEntity - What You Need for Viewport
```swift
public class CameraEntity: Entity {
    // Just the camera basics - not a full camera system
    @Published public var fov: Float = 60.0
    @Published public var nearPlane: Float = 0.1
    @Published public var farPlane: Float = 1000.0
    
    // That's it! Add more when you need it
}
```

### LightEntity - Basic Lighting
```swift
public class LightEntity: Entity {
    public enum LightType: String, Codable, CaseIterable {
        case directional, point, spot  // area comes later if needed
    }
    
    @Published public var lightType: LightType
    @Published public var intensity: Float = 1000.0
    @Published public var color: SIMD3<Float> = [1, 1, 1]
    
    // Not wrapped yet: shadows, cookies, advanced settings
    // Access via: entity.realityEntity.components[LightComponent.self]
}
```

### ModelEntity - 3D Model Loading
```swift
public class ModelEntity: Entity {
    @Published public var modelURL: URL?
    @Published public var isLoading: Bool = false
    
    public func load(from url: URL) async {
        // Simple async loading - that's all we need now
    }
    
    // Not wrapped: LOD, animation, materials
    // Access via: entity.realityEntity when needed
}
```

## The SceneEntity Protocol

```swift
public protocol SceneEntity: AnyObject, ObservableObject, Identifiable {
    // Minimal required interface
    var id: UUID { get }
    var name: String { get set }
    var position: SIMD3<Float> { get set }
    var rotation: simd_quatf { get set }
    var scale: SIMD3<Float> { get set }
    var entityType: SceneEntityType { get }
    
    // Bridge to full power
    var realityEntity: RealityKit.Entity { get }
}
```

**Why a protocol?**
- Allows different entity implementations
- Enables mock entities for testing
- Provides type-safe collections
- Keeps door open for alternative entity systems

## Using RealityKit Components Directly

When your simplified wrapper doesn't have what you need, go direct:

```swift
// Your wrapper provides basics
entity.position = SIMD3<Float>(1, 2, 3)  // Easy!

// Need something advanced? Use the escape hatch:
entity.realityEntity.components.set(
    CollisionComponent(shapes: [.generateBox(size: [1,1,1])])
)

// Access any RealityKit component
if let model = entity.realityEntity.components[ModelComponent.self] {
    // Full RealityKit power when needed
}

// Future: When collision is commonly used, wrap it
// entity.enableCollision(shape: .box)  // Coming in v3.0 maybe
```

## Entity Lifecycle

### Creation - Keep It Simple
```swift
// Basic entity creation
let entity = Entity(name: "MyEntity")

// Typed entities for common cases
let camera = CameraEntity(name: "MainCamera")
let light = LightEntity(name: "KeyLight", type: .directional)
let model = ModelEntity(name: "Character")

// Future: Factory methods added as needed
// let player = Entity.player()  // When you need player prefabs
```

### Scene Integration
```swift
// Add to scene
sceneManager.addEntity(entity)

// What happens (simplified):
// 1. Add to scene array
// 2. Add realityEntity to viewport
// 3. Make it selectable
// That's it! No complex systems yet
```

### Transform Operations - Unity-Like Simplicity
```swift
// What's wrapped (the 90% use case)
entity.position = SIMD3<Float>(0, 1, 0)
entity.rotation = simd_quatf(angle: .pi/4, axis: [0, 1, 0])
entity.scale = SIMD3<Float>(2, 2, 2)

// What's NOT wrapped yet
// entity.velocity = ...  // Add when you need physics
// entity.lookAt(target)  // Add when you need it
// entity.worldPosition   // Add when you need it
```

## Growing the System - Examples

### Example: Adding Animation Support (When Needed)

```swift
// FUTURE: When you need animation, extend Entity
extension Entity {
    func playAnimation(_ name: String) {
        // Wrap RealityKit animation when you actually use it
        if let animation = try? realityEntity.loadAnimation(named: name) {
            realityEntity.playAnimation(animation)
        }
    }
}

// Until then, use escape hatch if needed rarely
entity.realityEntity.playAnimation(...)
```

### Example: Adding Physics (When Needed)

```swift
// FUTURE: When you need physics
extension Entity {
    var isPhysicsEnabled: Bool {
        get { realityEntity.components[PhysicsBodyComponent.self] != nil }
        set {
            if newValue {
                realityEntity.components.set(PhysicsBodyComponent())
            } else {
                realityEntity.components.remove(PhysicsBodyComponent.self)
            }
        }
    }
}
```

## RealityKit Bridge Pattern

### Clear Type Disambiguation

```swift
// Your simplified wrapper
let myEntity = Entity(name: "Simplified")
myEntity.position = .zero  // Simple API

// Apple's full entity
let rkEntity = RealityKit.Entity()
rkEntity.components.set(...)  // Full API

// The bridge
let wrapped = myEntity.realityEntity  // Access full power

// ViewportState uses RealityKit.Entity directly
viewportState.rootEntity.addChild(myEntity.realityEntity)
```

### Why ViewportState Uses RealityKit.Entity

```swift
class ViewportState {
    // Uses RealityKit.Entity for rendering
    let rootEntity = RealityKit.Entity()
    
    // Why? Because ViewportState is about rendering,
    // not about your simplified API
}

// Your entities bridge to it
sceneManager.entities.forEach { entity in
    viewportState.rootEntity.addChild(entity.realityEntity)
}
```

## Performance Considerations

### Current Optimizations (What's Built)

```swift
// Shared update timer
private static let sharedTimer = Timer.publish(every: 1/60, on: .main, in: .common)

// Simple batching
func updateAllPositions(_ delta: SIMD3<Float>) {
    entities.forEach { $0.position += delta }
}
```

### Future Optimizations (When Needed)

```swift
// Entity pooling - when you have many entities
// Frustum culling - when scenes get large  
// LOD system - when detail becomes expensive
// Spatial indexing - when you need fast lookups

// Not built yet because YAGNI (You Aren't Gonna Need It)
```

## Common Patterns

### The "Good Enough" Pattern
```swift
// Don't over-engineer
struct EntityFactory {
    // Just what you need now
    static func createLight() -> LightEntity {
        return LightEntity(name: "Light", type: .directional)
    }
    
    // Not this:
    // static func createLightWithShadowsAndCookiesAndVolumetrics(...) 
}
```

### The "Escape Hatch" Pattern
```swift
// Wrapper for common case
entity.position = newPosition

// Escape hatch for advanced case
entity.realityEntity.components.set(
    MyCustomComponent()  // When wrapper doesn't have it
)
```

### The "Grow When Needed" Pattern
```swift
// Start with basics
class Entity {
    var position: SIMD3<Float>
}

// User needs rotation? Add it:
class Entity {
    var position: SIMD3<Float>
    var rotation: simd_quatf  // Added in v1.1
}

// User needs physics? Add it:
class Entity {
    var position: SIMD3<Float>
    var rotation: simd_quatf
    var velocity: SIMD3<Float>  // Added in v2.0
}
```

## Migration from Node System

The Node system is completely gone. This is not a 1:1 replacement but a fresh start:

| Node System (OLD) | Entity System (NEW) | Philosophy |
|-------------------|---------------------|------------|
| Complex hierarchy | Simple wrapper | Start simple |
| Everything upfront | Grow as needed | YAGNI |
| Custom everything | RealityKit backed | Use platform |
| Synchronous | Async where needed | Modern Swift |

## Best Practices

### DO: Start Simple
```swift
// Good - Ship it!
class MyEntity: Entity {
    var health: Int = 100
}

// Over-engineering - Don't do this day 1
class MyEntity: Entity {
    var health: HealthComponent
    var stats: StatsComponent
    var buffs: BuffSystem
    // ... 50 more systems
}
```

### DO: Use the Escape Hatch
```swift
// Wrapper doesn't have it? No problem!
entity.realityEntity.components.set(
    ParticleEmitterComponent()  // Direct RealityKit
)
```

### DON'T: Wrap Everything
```swift
// Don't wrap what you don't use
// ❌ AudioComponent wrapper with 50 methods
// ✅ entity.realityEntity.components.set(AudioComponent()) when needed
```

### DO: Keep It Observable
```swift
// SwiftUI-friendly from day 1
@Published var position: SIMD3<Float>
// Not just: var position: SIMD3<Float>
```

## Future Growth Areas

### Near Term (As Needed)
- [ ] Animation basics (when objects need to move)
- [ ] Collision detection (when things need to hit)
- [ ] Audio triggers (when you need sound)
- [ ] Parent constraints (when you need attachments)

### Medium Term (If Needed)
- [ ] Physics simulation (if you build physics games)
- [ ] Particle effects (if you need visual effects)
- [ ] Network replication (if you go multiplayer)
- [ ] Procedural mesh (if you generate geometry)

### Long Term (Maybe Never)
- [ ] Full ECS wrapper parity
- [ ] Custom component system
- [ ] Entity archetypes
- [ ] Parallel systems

**Remember: These are possibilities, not commitments. Build what you need when you need it.**

## Known Limitations (And That's OK)

Current limitations that are **intentional**:
- No complex animation system (use realityEntity)
- No physics wrapper (use realityEntity)  
- No audio abstractions (use realityEntity)
- No networking (not needed yet)

These aren't bugs, they're features. The wrapper stays simple and you can always access RealityKit directly.

## Debugging

### Simple Debug Info
```swift
extension Entity {
    var debugInfo: String {
        """
        \(name) [\(entityType)]
        Position: \(position)
        Has \(children.count) children
        """
    }
}
```

### When You Need More
```swift
// Access full RealityKit debug info
entity.realityEntity.debugDescription
entity.realityEntity.components.count
```

## See Also
- **Architecture.md** - Overall design philosophy
- **ViewportState.md** - How entities integrate with rendering
- **SceneManager.md** - Entity lifecycle management
- **MetalRendering.md** - How entities are rendered

## Summary

The Entity system is a **pragmatic wrapper** that:
- ✅ **Starts simple** - Just transforms and basic types
- ✅ **Grows with needs** - Add features when you use them
- ✅ **Provides escape hatches** - Full RealityKit always available
- ✅ **Ships today** - Not perfect, but perfectly functional
- ✅ **Stays maintainable** - Less code = fewer bugs

This is not about building a complete Entity system on day one. It's about building exactly what you need to ship your alpha, with room to grow into whatever your game becomes.