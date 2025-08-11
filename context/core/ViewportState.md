# RealityViewport ViewportState Documentation

**Purpose**: Document the central ViewportState management system with Entity/ECS architecture  
**Version**: 2.0  
**Criticality**: Core architectural component - Uses RealityKit.Entity directly  
**Last Updated**: August 2025

## ⚠️ CRITICAL: Type Disambiguation

ViewportState uses `RealityKit.Entity` DIRECTLY, not the custom Entity wrapper class:

```swift
// ViewportState uses RealityKit types explicitly
let rootEntity = RealityKit.Entity()      // NOT custom Entity
let cameraEntity = PerspectiveCamera()    // RealityKit subclass
var lightEntities: [UUID: RealityKit.Entity] = [:]  // RealityKit entities
var currentGizmo: RealityKit.Entity?      // RealityKit entity
```

## Overview

ViewportState is the single source of truth for all 3D viewport rendering and interaction, managing RealityKit entities directly without going through the custom Entity wrapper layer. This provides clean separation between the UI entity system and the rendering entity system.

## Architecture Position

```
RealityViewportApp
└── ContentView
    └── ViewportView
        ├── ViewportState (ObservableObject) ← RealityKit.Entity Management
        │   ├── rootEntity: RealityKit.Entity
        │   ├── cameraEntity: PerspectiveCamera (RealityKit)
        │   ├── lightEntities: [UUID: RealityKit.Entity]
        │   └── currentGizmo: RealityKit.Entity?
        ├── CameraController (uses ViewportState)
        └── SceneManager (provides Entity wrappers)
```

## Core Implementation

### Current ViewportState Class
```swift
@MainActor
class ViewportState: ObservableObject {
    // RealityKit entities - NOT custom Entity wrappers
    let rootEntity = RealityKit.Entity()
    let cameraEntity = PerspectiveCamera()
    var lightEntities: [UUID: RealityKit.Entity] = [:]
    
    // Viewport properties
    var viewportSize: CGSize = .zero
    var mouseLocation: CGPoint = .zero
    var lastDragValue: CGSize = .zero
    @Published var needsUpdate: Bool = false
    
    // Gizmo tracking
    var currentGizmo: RealityKit.Entity?
    
    // Camera state
    var cameraDistance: Float = 10.0
    var cameraAzimuth: Float = 0.785  // 45 degrees
    var cameraElevation: Float = 0.523  // 30 degrees
    var cameraTarget: SIMD3<Float> = [0, 0, 0]
    
    // Interaction mode
    @Published var interactionMode: ViewportInteractionMode = .environment
}
```

## Camera Mathematics (CORRECTED)

### Spherical Coordinate System
```swift
func computeCameraPosition() -> SIMD3<Float> {
    // Note: Different from v1.0 documentation - this is the actual implementation
    let x = cameraDistance * sin(cameraAzimuth) * cos(cameraElevation)
    let y = cameraDistance * sin(cameraElevation)
    let z = cameraDistance * cos(cameraAzimuth) * cos(cameraElevation)
    return cameraTarget + SIMD3<Float>(x, y, z)
}
```

**Key Properties:**
- `cameraDistance`: Radius from target (zoom level) - default 10.0
- `cameraAzimuth`: Horizontal rotation angle - default 0.785 (45°)
- `cameraElevation`: Vertical rotation angle - default 0.523 (30°)
- `cameraTarget`: Point camera orbits around - default [0,0,0]

## Interaction Modes

### ViewportInteractionMode
```swift
enum ViewportInteractionMode: String, CaseIterable {
    case environment  // Camera/viewport manipulation
    case entity      // Entity selection and transformation
}
```

**Mode Behaviors:**
- **Environment Mode**: All gestures control camera (orbit, pan, zoom)
- **Entity Mode**: Gestures select/transform entities, camera mostly locked

## Entity Bridge Pattern

### How ViewportState Works with Custom Entities

ViewportState doesn't know about custom Entity wrappers. Instead:

1. **SceneManager** manages custom Entity instances
2. **SceneManager** provides RealityKit.Entity to ViewportState
3. **ViewportState** manages RealityKit.Entity directly

```swift
// SceneManager adds entity's RealityKit representation
func addEntityToViewport(_ entity: Entity) {
    let rkEntity = entity.realityEntity  // Get RealityKit.Entity
    viewportState.rootEntity.addChild(rkEntity)
    
    // Track if it's a light
    if entity is LightEntity {
        viewportState.lightEntities[entity.id] = rkEntity
    }
}
```

## State Management

### Published Properties
```swift
@Published var needsUpdate: Bool = false
@Published var interactionMode: ViewportInteractionMode = .environment
```

### Update Pattern
```swift
// Toggle needsUpdate to trigger RealityView updates
func triggerUpdate() {
    needsUpdate.toggle()
}
```

## Gizmo Management

### Current Gizmo Tracking
```swift
var currentGizmo: RealityKit.Entity?  // Stores active gizmo entity
```

### Gizmo Creation (via SceneManager)
```swift
// SceneManager creates gizmo for selected entity
func updateGizmoForSelection(_ entity: Entity?) {
    // Remove old gizmo
    viewportState.currentGizmo?.removeFromParent()
    viewportState.currentGizmo = nil
    
    if let entity = entity {
        // Create new gizmo (RealityKit.Entity)
        let gizmo = TransformGizmo.createGizmoEntity()
        gizmo.position = entity.position
        viewportState.rootEntity.addChild(gizmo)
        viewportState.currentGizmo = gizmo
    }
}
```

## Light Entity Management

### Light Tracking
```swift
var lightEntities: [UUID: RealityKit.Entity] = [:]
```

### Light Registration
```swift
// When adding a light entity
func registerLight(_ entityId: UUID, _ rkLight: RealityKit.Entity) {
    lightEntities[entityId] = rkLight
}

// When removing a light entity  
func unregisterLight(_ entityId: UUID) {
    lightEntities[entityId]?.removeFromParent()
    lightEntities.removeValue(forKey: entityId)
}
```

## Platform-Specific Adaptations

### Mouse/Trackpad State
```swift
var mouseLocation: CGPoint = .zero      // Current mouse position
var lastDragValue: CGSize = .zero       // For drag delta calculation
```

### Viewport Sizing
```swift
var viewportSize: CGSize = .zero  // Updated by ViewportView on resize
```

## Integration Points

### With RealityView
```swift
RealityView { content in
    content.add(viewportState.rootEntity)
    content.add(viewportState.cameraEntity)
} update: { _ in
    // Triggered by needsUpdate changes
}
```

### With SceneManager
```swift
// SceneManager bridges custom entities to ViewportState
class SceneManager {
    func syncToViewport() {
        for entity in entities {
            let rkEntity = entity.realityEntity
            viewportState.rootEntity.addChild(rkEntity)
        }
    }
}
```

### With CameraController
```swift
// CameraController modifies ViewportState directly
class CameraController {
    func updateCamera(azimuth: Float, elevation: Float) {
        viewportState.cameraAzimuth = azimuth
        viewportState.cameraElevation = elevation
        viewportState.cameraEntity.position = viewportState.computeCameraPosition()
    }
}
```

## Critical Distinctions from v1.0

| v1.0 Documentation | v2.0 Reality |
|-------------------|--------------|
| References Node system | Uses Entity system |
| Custom Entity refs | RealityKit.Entity only |
| Wrong camera math | Corrected sin/cos order |
| Gizmo methods shown | Gizmo managed externally |
| Generic patterns | Actual implementation |

## Best Practices

### DO: Maintain Type Clarity
```swift
// Always be explicit about which Entity type
let customEntity = Entity()                    // Custom wrapper
let rkEntity = customEntity.realityEntity      // RealityKit.Entity
viewportState.rootEntity.addChild(rkEntity)    // Uses RealityKit type
```

### DON'T: Mix Entity Types
```swift
// WRONG - Type mismatch
viewportState.rootEntity = Entity()  // Error: expects RealityKit.Entity

// CORRECT
viewportState.rootEntity.addChild(entity.realityEntity)
```

### DO: Use ViewportState for RealityKit Operations
```swift
// ViewportState manages RealityKit entities
viewportState.cameraEntity.position = newPosition
viewportState.rootEntity.scale = newScale
```

### DON'T: Store Custom Entities in ViewportState
```swift
// WRONG - ViewportState doesn't know about custom Entity
viewportState.selectedEntity = myEntity  // No such property

// CORRECT - Selection is in SelectionManager
selectionManager.select(myEntity)
```

## Common Operations

### Camera Reset
```swift
func resetCamera() {
    viewportState.cameraDistance = 10.0
    viewportState.cameraAzimuth = 0.785
    viewportState.cameraElevation = 0.523
    viewportState.cameraTarget = [0, 0, 0]
    viewportState.cameraEntity.position = viewportState.computeCameraPosition()
    viewportState.cameraEntity.look(at: viewportState.cameraTarget, from: viewportState.cameraEntity.position, relativeTo: nil)
}
```

### Mode Toggle
```swift
func toggleInteractionMode() {
    viewportState.interactionMode = 
        viewportState.interactionMode == .environment ? .entity : .environment
    
    // Update gizmo visibility
    if viewportState.interactionMode == .environment {
        viewportState.currentGizmo?.isEnabled = false
    } else {
        viewportState.currentGizmo?.isEnabled = true
    }
}
```

### Update Trigger
```swift
// Force RealityView update
viewportState.needsUpdate.toggle()
```

## Performance Considerations

- **Direct RealityKit Access**: No wrapper overhead
- **Minimal Publishing**: Only two @Published properties
- **Efficient Updates**: Toggle pattern for needsUpdate
- **Lazy Gizmo Creation**: Only when entity selected
- **Light Tracking**: O(1) lookup via dictionary

## Migration Notes (v1.0 → v2.0)

### What Changed
- All Node references → Entity references
- Custom Entity storage → RealityKit.Entity only
- Camera math corrected (sin/cos order)
- Gizmo creation moved to external managers
- Light entity tracking added

### Breaking Changes
- No selectedEntity property (use SelectionManager)
- No Entity wrapper storage (use SceneManager)
- Camera computation function different
- Gizmo lifecycle managed externally

## Future Enhancements

- [ ] Multiple viewport support
- [ ] Viewport overlay system
- [ ] Performance metrics tracking
- [ ] Camera bookmarks
- [ ] Grid configuration in state
- [ ] Debug visualization options

## Summary

ViewportState v2.0 is a focused, type-safe manager for RealityKit entities that:
- ✅ Uses RealityKit.Entity directly (no custom Entity wrappers)
- ✅ Provides clean separation from UI entity system
- ✅ Manages camera, lights, and gizmos efficiently
- ✅ Integrates cleanly with SceneManager bridge pattern
- ✅ Maintains high performance with minimal overhead