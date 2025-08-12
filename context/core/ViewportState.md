# RealityViewport ViewportState Documentation

**Module**: VIEWPORTSTATE.md  
**Version**: 3.0  
**Architecture**: RealityKit.Entity Bridge  
**Philosophy**: Clean separation between wrapper and rendering  
**Status**: Core System - 95% Complete  
**Last Updated**: August 2025

## Overview

ViewportState is the **rendering bridge** - it manages RealityKit entities directly for viewport rendering while your simplified Entity wrappers handle the API layer. This separation is intentional and clean.

## Core Philosophy

**"ViewportState handles rendering. Entity wrappers handle API. Never mix them."**

```swift
// Your simplified Entity wrapper (API layer)
let entity = Entity()
entity.position = SIMD3<Float>(1, 2, 3)  // Simple API

// ViewportState (rendering layer)
viewportState.rootEntity.addChild(entity.realityEntity)  // Bridge to rendering
```

## Why This Separation?

```yaml
Entity Wrapper (Your Code):
  Purpose: Simple API for users
  Scope: What you need today
  Growth: Expands as needed
  
ViewportState (This Class):
  Purpose: Manage RealityKit rendering
  Scope: Camera, lights, scene graph
  Type: RealityKit.Entity only
  
The Bridge:
  entity.realityEntity connects them
  Clean separation of concerns
  No type confusion
```

## Type Disambiguation

### Standard Pattern
```swift
// ALWAYS use this pattern to clarify Entity types:

// Your simplified Entity wrapper
import RealityViewport
let myEntity = Entity()                    // Your simplified wrapper
myEntity.position = SIMD3<Float>(1, 2, 3)  // Simple API

// ViewportState uses RealityKit directly
let viewportState = ViewportState()
viewportState.rootEntity                   // RealityKit.Entity (NOT your Entity)
viewportState.cameraEntity                 // PerspectiveCamera (RealityKit type)

// The bridge between them
viewportState.rootEntity.addChild(myEntity.realityEntity)  // .realityEntity bridges
```

## Core Implementation

Based on actual ViewportState.swift:

```swift
@MainActor
class ViewportState: ObservableObject {
    // === RENDERING ENTITIES (RealityKit types) ===
    let rootEntity = RealityKit.Entity()        // Scene root
    let cameraEntity = PerspectiveCamera()      // RealityKit camera
    var lightEntities: [UUID: RealityKit.Entity] = [:]  // Light tracking
    
    // === VIEWPORT PROPERTIES ===
    var viewportSize: CGSize = .zero            // Current size
    var mouseLocation: CGPoint = .zero          // Mouse position
    var lastDragValue: CGSize = .zero           // Drag tracking
    @Published var needsUpdate: Bool = false    // Update trigger
    
    // === GIZMO ===
    var currentGizmo: RealityKit.Entity?        // Transform gizmo
    
    // === CAMERA STATE ===
    var cameraDistance: Float = 10.0            // Zoom
    var cameraAzimuth: Float = 0.785            // Horizontal rotation (45°)
    var cameraElevation: Float = 0.523          // Vertical rotation (30°)
    var cameraTarget: SIMD3<Float> = [0, 0, 0]  // Orbit center
    
    // === INTERACTION MODE ===
    enum ViewportInteractionMode: String, CaseIterable {
        case environment  // Camera control
        case entity      // Entity manipulation
    }
    @Published var interactionMode: ViewportInteractionMode = .environment
}
```

## The Bridge Pattern

### How Your Entities Connect to ViewportState

```swift
// Your Entity wrapper provides simplified API
class Entity {
    @Published var position: SIMD3<Float>
    let realityEntity: RealityKit.Entity  // The bridge!
}

// SceneManager connects them
class SceneManager {
    func addEntity(_ entity: Entity) {
        // Your wrapper for API
        entities.append(entity)
        
        // Bridge to rendering
        viewportState.rootEntity.addChild(entity.realityEntity)
        
        // Special handling for lights
        if let light = entity as? LightEntity {
            viewportState.lightEntities[entity.id] = entity.realityEntity
        }
    }
}
```

### Why This Pattern Works

```yaml
Benefits:
  - Type Safety: No confusion between Entity types
  - Clean Layers: API separate from rendering
  - Performance: Direct RealityKit access
  - Flexibility: Can change wrapper without touching rendering
  - Growth: Wrapper grows, rendering stays stable
```

## Camera System

### Spherical Coordinate Mathematics
```swift
func computeCameraPosition() -> SIMD3<Float> {
    // Convert spherical to Cartesian coordinates
    let x = cameraDistance * sin(cameraAzimuth) * cos(cameraElevation)
    let y = cameraDistance * sin(cameraElevation)
    let z = cameraDistance * cos(cameraAzimuth) * cos(cameraElevation)
    return cameraTarget + SIMD3<Float>(x, y, z)
}
```

### Camera Properties Explained
```yaml
cameraDistance:
  Type: Float
  Default: 10.0
  Purpose: How far from target (zoom)
  
cameraAzimuth:
  Type: Float (radians)
  Default: 0.785 (45°)
  Purpose: Horizontal orbit angle
  
cameraElevation:
  Type: Float (radians)
  Default: 0.523 (30°)
  Purpose: Vertical orbit angle
  
cameraTarget:
  Type: SIMD3<Float>
  Default: [0, 0, 0]
  Purpose: Point camera orbits around
```

## Interaction Modes

### Mode-Based Input Routing
```swift
enum ViewportInteractionMode {
    case environment  // All input controls camera
    case entity      // Input controls selected entity
}

// Usage pattern
switch viewportState.interactionMode {
case .environment:
    // Drag orbits camera
    cameraController.handleOrbit(dragDelta)
case .entity:
    // Drag moves selected entity
    if let selected = selectionManager.selectedEntity {
        selected.position += dragDelta
    }
}
```

## State Management

### Minimal Published Properties
```swift
// Only 2 @Published for performance
@Published var needsUpdate: Bool = false
@Published var interactionMode: ViewportInteractionMode = .environment
```

### Update Trigger Pattern
```swift
// Efficient update triggering
func triggerViewportUpdate() {
    viewportState.needsUpdate.toggle()  // Toggle triggers update
}

// In RealityView
RealityView { content in
    // Initial setup
} update: { _ in
    // Called when needsUpdate changes
}
```

## Light Management

### Why Track Lights Separately?
```swift
var lightEntities: [UUID: RealityKit.Entity] = [:]

// Lights need special handling for:
// - Day/night cycle updates
// - Shadow configuration
// - Performance optimization
// - Scene lighting calculations
```

### Light Registration Pattern
```swift
// Adding a light
func registerLight(_ entity: LightEntity) {
    let rkLight = entity.realityEntity
    viewportState.lightEntities[entity.id] = rkLight
    viewportState.rootEntity.addChild(rkLight)
}

// Removing a light
func unregisterLight(_ entityId: UUID) {
    if let light = viewportState.lightEntities[entityId] {
        light.removeFromParent()
        viewportState.lightEntities.removeValue(forKey: entityId)
    }
}
```

## Gizmo Management

### Gizmo Lifecycle
```swift
var currentGizmo: RealityKit.Entity?  // Only one gizmo at a time

// Create gizmo for selected entity
func showGizmo(for entity: Entity) {
    // Remove old gizmo
    currentGizmo?.removeFromParent()
    
    // Create new gizmo at entity position
    let gizmo = TransformGizmo.create()  // Returns RealityKit.Entity
    gizmo.position = entity.position
    rootEntity.addChild(gizmo)
    currentGizmo = gizmo
}

// Hide gizmo
func hideGizmo() {
    currentGizmo?.removeFromParent()
    currentGizmo = nil
}
```

## Common Patterns

### Pattern: Adding Entities
```swift
// Your wrapper entity
let entity = ModelEntity(name: "Box")

// Bridge to viewport
viewportState.rootEntity.addChild(entity.realityEntity)

// Never do this - wrong type!
// viewportState.rootEntity.addChild(entity)  // ❌ Type error
```

### Pattern: Camera Control
```swift
// Orbit camera
func orbitCamera(deltaX: Float, deltaY: Float) {
    viewportState.cameraAzimuth += deltaX * 0.01
    viewportState.cameraElevation += deltaY * 0.01
    
    // Update position
    let position = viewportState.computeCameraPosition()
    viewportState.cameraEntity.position = position
    viewportState.cameraEntity.look(at: viewportState.cameraTarget, 
                                   from: position, 
                                   relativeTo: nil)
}
```

### Pattern: Mode Switching
```swift
// Toggle between camera and entity control
func toggleMode() {
    viewportState.interactionMode = 
        viewportState.interactionMode == .environment ? .entity : .environment
    
    // Update UI accordingly
    if viewportState.interactionMode == .entity {
        showGizmo(for: selectedEntity)
    } else {
        hideGizmo()
    }
}
```

## Integration Examples

### With RealityView
```swift
struct ViewportView: View {
    @StateObject var viewportState = ViewportState()
    
    var body: some View {
        RealityView { content in
            // Add viewport entities
            content.add(viewportState.rootEntity)
            content.add(viewportState.cameraEntity)
        } update: { _ in
            // Triggered by needsUpdate changes
        }
        .onChange(of: viewportState.needsUpdate) { _ in
            // React to updates
        }
    }
}
```

### With SceneManager
```swift
class SceneManager {
    @Published var entities: [Entity] = []
    let viewportState: ViewportState
    
    func syncToViewport() {
        // Clear viewport
        viewportState.rootEntity.children.removeAll()
        
        // Add all entities via bridge
        for entity in entities {
            viewportState.rootEntity.addChild(entity.realityEntity)
        }
    }
}
```

## Performance Considerations

### Why Direct RealityKit?
```yaml
No Wrapper Overhead:
  - Direct RealityKit.Entity manipulation
  - No property syncing needed
  - Native performance
  
Minimal Publishing:
  - Only 2 @Published properties
  - Reduces SwiftUI updates
  - Better frame rates
  
Efficient Updates:
  - Toggle pattern for needsUpdate
  - Batch changes before triggering
  - One update per frame max
```

## Common Mistakes to Avoid

### ❌ DON'T: Mix Entity Types
```swift
// WRONG - Type mismatch
viewportState.rootEntity = Entity()  // Can't assign wrapper to RealityKit.Entity

// CORRECT - Use the bridge
viewportState.rootEntity.addChild(entity.realityEntity)
```

### ❌ DON'T: Store Wrappers in ViewportState
```swift
// WRONG - ViewportState doesn't know about wrappers
viewportState.selectedEntity = myEntity  // No such property

// CORRECT - Selection is elsewhere
selectionManager.selectedEntity = myEntity
```

### ❌ DON'T: Forget the Bridge
```swift
// WRONG - Adding wrapper directly
viewportState.rootEntity.addChild(myEntity)  // Type error

// CORRECT - Use realityEntity property
viewportState.rootEntity.addChild(myEntity.realityEntity)
```

## Future Enhancements

### Planned (When Needed)
- [ ] Multiple viewport support (split views)
- [ ] Camera bookmarks (saved positions)
- [ ] Performance metrics overlay
- [ ] Viewport effects (fog, bloom)

### Maybe Never (YAGNI)
- [ ] Complex viewport layouts
- [ ] Multiple camera types
- [ ] Advanced render settings
- [ ] Post-processing pipeline

## See Also
- **EntitySystem.md** - Your simplified Entity wrapper
- **Architecture.md** - Bridge pattern explanation
- **SceneManager.md** - How entities connect to viewport
- **CameraController.md** - Camera manipulation details

## Summary

ViewportState is the **clean rendering bridge** that:
- ✅ **Uses RealityKit.Entity directly** - No wrapper overhead
- ✅ **Bridges via .realityEntity** - Clean connection point
- ✅ **Separates concerns** - Rendering vs API
- ✅ **Stays focused** - Just viewport management
- ✅ **Performs well** - Minimal overhead, direct access

This separation between your simplified Entity wrapper (API) and ViewportState (rendering) is intentional and powerful. It lets your wrapper grow while keeping rendering stable and performant.