# RealityViewport ViewportState Documentation

**Purpose**: Document the central ViewportState management system  
**Version**: 1.0  
**Criticality**: Core architectural component  
**Last Updated**: July 2025

## Overview

ViewportState is the single source of truth for all 3D viewport rendering and interaction. It orchestrates camera positioning, interaction modes, gizmo management, and RealityKit entity lifecycle.

## Architecture Position

```
RealityViewportApp
└── ContentView
    └── ViewportView
        ├── ViewportState (ObservableObject) ← Central State
        │   ├── Camera Management
        │   ├── Interaction Mode Control
        │   ├── Gizmo Lifecycle
        │   └── Entity References
        ├── CameraController (uses ViewportState)
        └── TransformGizmo (rendered via ViewportState)
```

## Core Components

### ViewportInteractionMode
```swift
enum ViewportInteractionMode: String, CaseIterable {
    case environment = "Environment"  // Camera manipulation
    case entity = "Entity"           // Object selection/transformation
}
```

**Mode Behaviors:**
- **Environment Mode**: All gestures control camera (orbit, pan, zoom)
- **Entity Mode**: Gestures select/transform objects, camera mostly locked

### Camera Mathematics

**Spherical Coordinate System:**
```swift
class ViewportState: ObservableObject {
    @Published var cameraDistance: Float = 5.0
    @Published var cameraRotation: SIMD2<Float> = .zero
    @Published var cameraTarget: SIMD3<Float> = .zero
    
    // Spherical to Cartesian conversion
    var cameraPosition: SIMD3<Float> {
        let x = cameraDistance * cos(cameraRotation.y) * sin(cameraRotation.x)
        let y = cameraDistance * sin(cameraRotation.y)
        let z = cameraDistance * cos(cameraRotation.y) * cos(cameraRotation.x)
        return cameraTarget + SIMD3(x, y, z)
    }
}
```

**Key Properties:**
- `cameraDistance`: Radius from target (zoom level)
- `cameraRotation`: X=azimuth, Y=elevation angles
- `cameraTarget`: Point camera orbits around

## State Management

### Published Properties
```swift
@Published var needsSceneUpdate = false
@Published var currentMode: ViewportInteractionMode = .environment
@Published var selectedEntity: Entity?
@Published var gizmoEntity: Entity?
@Published var isGizmoVisible = true
```

### Update Trigger Pattern
```swift
// Any significant state change triggers scene update
private func triggerUpdate() {
    needsSceneUpdate.toggle()
}
```

## Gizmo Lifecycle

### Gizmo Creation
```swift
func createGizmo(for entity: Entity) {
    gizmoEntity?.removeFromParent()
    
    let newGizmo = TransformGizmo.createGizmoEntity()
    newGizmo.position = entity.position
    scene.addChild(newGizmo)
    
    gizmoEntity = newGizmo
}
```

### Gizmo Visibility Rules
- Hidden in Environment mode
- Shown when entity selected in Entity mode
- Scales with camera distance for consistent size
- Updates position to match selected entity

## Entity Management

### Entity References
```swift
// Core scene entities managed by ViewportState
var cameraEntity: PerspectiveCamera?
var gridEntity: Entity?
var axisHelperEntity: Entity?
```

### Entity Lifecycle
1. **Creation**: Entities created on viewport initialization
2. **Updates**: Transform updates through ViewportState only
3. **Cleanup**: Proper removal and nil assignment

## Platform-Specific Adaptations

### macOS
- Precise mouse coordinates for gizmo interaction
- Scroll wheel zoom with momentum
- Right-click for context menus

### iOS
- Touch-friendly gizmo hit areas (44pt minimum)
- Pinch zoom with natural feel
- Long press for selection options

### tvOS
- Focus-based entity selection
- Remote swipe for camera orbit
- Play/pause for mode toggle

## Integration Points

### With CameraController
```swift
// CameraController reads ViewportState
let position = viewportState.cameraPosition
let target = viewportState.cameraTarget

// But updates through methods
viewportState.updateCamera(rotation: newRotation)
```

### With SelectionManager
```swift
// Selection changes update ViewportState
selectionManager.$selectedEntity
    .sink { entity in
        viewportState.selectedEntity = entity
        viewportState.updateGizmo()
    }
```

### With SceneManager
```swift
// Scene changes trigger viewport updates
sceneManager.$rootEntity
    .sink { _ in
        viewportState.triggerUpdate()
    }
```

## Best Practices

### DO: Use ViewportState Methods
```swift
// Correct: Use provided methods
viewportState.setInteractionMode(.entity)
viewportState.focusOnEntity(modelNode.entity)
```

### DON'T: Direct Property Mutation
```swift
// Wrong: Direct mutation from views
viewportState.cameraDistance = 10 // From random view
```

### DO: Observe needsSceneUpdate
```swift
// React to scene changes
viewportState.$needsSceneUpdate
    .sink { _ in
        renderScene()
    }
```

## Common Patterns

### Mode Switching
```swift
Button(action: {
    viewportState.setInteractionMode(
        viewportState.currentMode == .environment ? .entity : .environment
    )
}) {
    Image(systemName: viewportState.currentMode.iconName)
}
```

### Gizmo Interaction
```swift
// In gesture handler
if viewportState.currentMode == .entity,
   let hit = hitTest(location),
   hit.entity == viewportState.gizmoEntity {
    // Handle gizmo drag
}
```

### Camera Reset
```swift
func resetCamera() {
    withAnimation(.easeInOut(duration: 0.5)) {
        viewportState.cameraDistance = 5.0
        viewportState.cameraRotation = .zero
        viewportState.cameraTarget = .zero
    }
}
```

## Performance Considerations

- **Update Batching**: Multiple state changes trigger single render
- **Lazy Gizmo Creation**: Only create when needed
- **Efficient Hit Testing**: Cache hit test results during drag
- **Minimal Publishing**: Only publish changes that affect UI

## Future Enhancements

- [ ] Multiple viewport support
- [ ] Saved camera positions
- [ ] Viewport overlays (stats, grid options)
- [ ] Custom gizmo styles
- [ ] Animation system integration