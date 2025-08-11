# RealityViewport Gesture & Input Documentation

**Purpose**: Document sophisticated gesture and input handling with Entity/ECS system  
**Version**: 2.0  
**Complexity**: Platform-specific with unified behavior  
**Last Updated**: August 2025

## Overview

RealityViewport implements a mode-based gesture system that adapts to each platform while maintaining consistent behavior. Input is routed through ViewportState modes to handle Entity manipulation or camera control.

## Mode-Based Gesture Routing (Entity System)

### Input Flow
```
User Input → Platform Gesture → ViewportState.interactionMode → Handler
                                        ↓
                              .environment → CameraController
                                        ↓
                                .entity → SelectionManager/Entity Transform
```

### Mode Definitions
```swift
enum ViewportInteractionMode {
    case environment  // Camera manipulation
    case entity      // Entity selection/transformation
    
    func handleDrag(_ translation: CGSize, viewport: ViewportView, 
                   sceneManager: SceneManager) {
        switch self {
        case .environment:
            viewport.cameraController.handleOrbitDrag(translation)
        case .entity:
            viewport.handleEntityDrag(translation, sceneManager: sceneManager)
        }
    }
}
```

## Platform-Specific Input Differences

### macOS Input Mapping (Entity-Aware)
```swift
// Primary Input
.gesture(
    DragGesture(minimumDistance: 1)
        .onChanged { value in
            if viewportState.interactionMode == .environment {
                // Left drag = orbit
                cameraController.orbit(by: value.translation)
            } else {
                // Left drag = select/move entity
                if let entity = hitTestEntity(at: value.location) {
                    selectionManager.select(entity)
                    handleEntityTransform(entity, delta: value.translation)
                }
            }
        }
)

// Secondary Input (Right-click)
.gesture(
    DragGesture(minimumDistance: 1)
        .modifiers(.option)  // Right-click simulation
        .onChanged { value in
            // Always pan camera
            cameraController.pan(by: value.translation)
        }
)

// Zoom
.onScroll { delta in
    cameraController.zoom(by: delta.y)
}
```

### iOS Input Mapping (Entity-Aware)
```swift
// Single Touch
.gesture(
    DragGesture(minimumDistance: 10)
        .onChanged { value in
            if viewportState.interactionMode == .entity {
                handleEntityInteraction(at: value.location)
            } else {
                cameraController.orbit(by: value.translation)
            }
        }
)

// Two Finger Pan
.gesture(
    DragGesture(minimumDistance: 10)
        .simultaneously(with: TapGesture(count: 2))
        .onChanged { value in
            cameraController.pan(by: value.translation)
        }
)

// Pinch Zoom
.gesture(
    MagnificationGesture()
        .onChanged { scale in
            cameraController.zoom(to: scale)
        }
)

// Long Press for Entity Context Menu
.gesture(
    LongPressGesture(minimumDuration: 0.5)
        .onEnded { _ in
            if let entity = hitTestEntity(at: location) {
                showEntityContextMenu(for: entity)
            }
        }
)
```

### tvOS Input Mapping (Entity Support)
```swift
// Siri Remote Swipes
.onMoveCommand { direction in
    if viewportState.interactionMode == .environment {
        switch direction {
        case .left, .right:
            cameraController.orbit(horizontal: direction == .left ? -1 : 1)
        case .up, .down:
            cameraController.orbit(vertical: direction == .up ? 1 : -1)
        }
    } else {
        // Navigate between entities
        navigateEntities(direction: direction)
    }
}

// Play/Pause Button
.onPlayPauseCommand {
    viewportState.interactionMode = 
        viewportState.interactionMode == .environment ? .entity : .environment
}

// Select Button (Entity Mode)
.onSelectCommand {
    if viewportState.interactionMode == .entity {
        if let focusedEntity = focusedEntity {
            selectionManager.select(focusedEntity)
        }
    }
}
```

## Entity Hit Detection

### Hit Test Implementation (RealityKit.Entity)
```swift
func hitTestEntity(at location: CGPoint) -> (any SceneEntity)? {
    // Get ray from camera
    guard let ray = arView.ray(through: location) else { return nil }
    
    // Hit test against RealityKit entities
    let hits = scene.raycast(
        origin: ray.origin,
        direction: ray.direction,
        length: 100,
        query: .nearest,
        mask: .default
    )
    
    // Map RealityKit.Entity to custom Entity
    if let hit = hits.first {
        let rkEntity = hit.entity  // This is RealityKit.Entity
        
        // Find corresponding custom Entity in SceneManager
        return sceneManager.findEntity(byRealityEntity: rkEntity)
    }
    
    return nil
}
```

### Entity Selection States
```swift
enum EntityInteractionState {
    case idle
    case hovering(entity: any SceneEntity)
    case selected(entity: any SceneEntity)
    case transforming(entity: any SceneEntity, gizmoAxis: GizmoAxis)
    
    var cursorStyle: NSCursor? {
        switch self {
        case .idle:
            return .arrow
        case .hovering:
            return .pointingHand
        case .selected:
            return .openHand
        case .transforming:
            return .closedHand
        }
    }
}
```

## Gizmo Interaction with Entities

### Gizmo Hit Detection (Entity-Based)
```swift
func hitTestGizmo(at location: CGPoint) -> GizmoAxis? {
    // Gizmo is a RealityKit.Entity in ViewportState
    guard let gizmoEntity = viewportState.currentGizmo else { return nil }
    
    let ray = viewport.ray(from: location)
    
    // Test against gizmo components
    if let hit = scene.raycast(
        origin: ray.origin,
        direction: ray.direction,
        length: 100,
        query: .nearest,
        mask: .gizmo
    ).first {
        // Check if hit entity is part of gizmo
        if isGizmoComponent(hit.entity, of: gizmoEntity) {
            return gizmoAxis(for: hit.entity)
        }
    }
    
    return nil
}
```

### Axis-Constrained Entity Movement
```swift
func dragEntity(entity: any SceneEntity, axis: GizmoAxis, by delta: CGSize) {
    // Convert screen delta to world space
    let worldDelta = screenToWorld(delta)
    
    // Apply constraint to entity position
    let movement: SIMD3<Float>
    switch axis {
    case .x:
        movement = SIMD3(worldDelta.x, 0, 0)
    case .y:
        movement = SIMD3(0, worldDelta.y, 0)
    case .z:
        movement = SIMD3(0, 0, worldDelta.z)
    case .xy:
        movement = SIMD3(worldDelta.x, worldDelta.y, 0)
    case .xz:
        movement = SIMD3(worldDelta.x, 0, worldDelta.z)
    case .yz:
        movement = SIMD3(0, worldDelta.y, worldDelta.z)
    case .xyz:
        movement = worldDelta
    }
    
    // Update entity position
    entity.position += movement
    
    // Sync gizmo position
    viewportState.currentGizmo?.position = entity.position
    
    // Trigger viewport update
    viewportState.needsUpdate.toggle()
}
```

## Camera Control Mathematics (Updated)

### Orbit Calculation
```swift
func orbitCamera(by delta: CGSize) {
    // Convert pixel movement to radians
    let sensitivity: Float = 0.01
    let deltaX = Float(delta.width) * sensitivity
    let deltaY = Float(delta.height) * sensitivity
    
    // Update spherical coordinates
    viewportState.cameraAzimuth += deltaX
    viewportState.cameraElevation += deltaY
    
    // Clamp elevation to prevent flipping
    viewportState.cameraElevation = viewportState.cameraElevation
        .clamped(to: -.pi/2 + 0.1...pi/2 - 0.1)
    
    // Update camera position using ViewportState's method
    let newPosition = viewportState.computeCameraPosition()
    viewportState.cameraEntity.position = newPosition
    viewportState.cameraEntity.look(
        at: viewportState.cameraTarget,
        from: newPosition,
        relativeTo: nil
    )
}
```

### Pan Calculation
```swift
func panCamera(by delta: CGSize) {
    // Get camera right and up vectors
    let transform = viewportState.cameraEntity.transform
    let right = transform.matrix.columns.0.xyz
    let up = transform.matrix.columns.1.xyz
    
    // Scale by distance for consistent movement
    let scale = viewportState.cameraDistance * 0.001
    let deltaX = Float(delta.width) * scale
    let deltaY = Float(delta.height) * scale
    
    // Move target
    viewportState.cameraTarget += right * deltaX
    viewportState.cameraTarget += up * -deltaY
    
    // Update camera to look at new target
    let position = viewportState.computeCameraPosition()
    viewportState.cameraEntity.position = position
    viewportState.cameraEntity.look(
        at: viewportState.cameraTarget,
        from: position,
        relativeTo: nil
    )
}
```

### Zoom Calculation
```swift
func zoomCamera(by delta: Float) {
    // Exponential zoom for natural feel
    let zoomSpeed: Float = 0.1
    let factor = exp(-delta * zoomSpeed)
    
    viewportState.cameraDistance *= factor
    
    // Clamp to reasonable range
    viewportState.cameraDistance = viewportState.cameraDistance
        .clamped(to: 0.5...50.0)
    
    // Update camera position
    let position = viewportState.computeCameraPosition()
    viewportState.cameraEntity.position = position
}
```

## Touch Target Optimization (Entity Selection)

### iOS Touch Areas for Entities
```swift
struct TouchTargets {
    static let minimumTapTarget: CGFloat = 44  // Apple HIG
    static let gizmoHitPadding: CGFloat = 20   // Extra hit area
    static let entitySelectPadding: CGFloat = 10
}

func expandedEntityHitTest(at point: CGPoint) -> (any SceneEntity)? {
    // Test expanded area for better touch accuracy
    let testPoints = [
        point,
        point.offset(dx: TouchTargets.entitySelectPadding, dy: 0),
        point.offset(dx: -TouchTargets.entitySelectPadding, dy: 0),
        point.offset(dx: 0, dy: TouchTargets.entitySelectPadding),
        point.offset(dx: 0, dy: -TouchTargets.entitySelectPadding)
    ]
    
    for testPoint in testPoints {
        if let entity = hitTestEntity(at: testPoint) {
            return entity
        }
    }
    
    return nil
}
```

## Gesture Conflict Resolution

### Entity vs Camera Priority
```swift
// Entity selection takes priority in entity mode
struct GesturePriority {
    static let entitySelection = 1000
    static let gizmoInteraction = 900
    static let cameraOrbit = 800
    static let cameraPan = 700
    static let cameraZoom = 600
}

// Apply priorities
entityTapGesture.priority(GesturePriority.entitySelection)
orbitGesture.priority(GesturePriority.cameraOrbit)
```

## Haptic Feedback for Entity Operations (iOS)

### Entity Feedback Patterns
```swift
enum EntityHapticPattern {
    case entitySelected
    case entityCreated
    case entityDeleted
    case transformStart
    case transformSnap
    case modeSwitch
    
    func trigger() {
        #if os(iOS)
        switch self {
        case .entitySelected:
            UIImpactFeedbackGenerator(style: .light).impactOccurred()
        case .entityCreated:
            UIImpactFeedbackGenerator(style: .medium).impactOccurred()
        case .entityDeleted:
            UINotificationFeedbackGenerator().notificationOccurred(.warning)
        case .transformStart:
            UIImpactFeedbackGenerator(style: .light).impactOccurred()
        case .transformSnap:
            UIImpactFeedbackGenerator(style: .rigid).impactOccurred()
        case .modeSwitch:
            UISelectionFeedbackGenerator().selectionChanged()
        }
        #endif
    }
}
```

## Keyboard Shortcuts for Entity Operations (macOS)

### Entity-Specific Shortcuts
```swift
extension View {
    func entityKeyboardShortcuts() -> some View {
        self
            // Transform tools
            .keyboardShortcut("w", modifiers: [])  // Move tool
            .keyboardShortcut("e", modifiers: [])  // Rotate tool
            .keyboardShortcut("r", modifiers: [])  // Scale tool
            .keyboardShortcut("q", modifiers: [])  // Select tool
            
            // Entity operations
            .keyboardShortcut("d", modifiers: .command)  // Duplicate entity
            .keyboardShortcut(.delete, modifiers: [])    // Delete entity
            .keyboardShortcut("f", modifiers: [])        // Focus entity
            .keyboardShortcut("h", modifiers: [])        // Hide entity
            
            // Mode switching
            .keyboardShortcut(.space, modifiers: [])     // Toggle mode
            
            // Scene operations
            .keyboardShortcut("g", modifiers: [])        // Toggle grid
            .keyboardShortcut("l", modifiers: .command)  // Toggle lights
    }
}
```

## Performance Optimizations

### Entity Hit Test Caching
```swift
struct EntityHitTestCache {
    private var cache: [CGPoint: (any SceneEntity)?] = [:]
    private var lastUpdate: Date = Date()
    private let maxAge: TimeInterval = 0.1
    
    mutating func hitTest(at point: CGPoint, 
                         using test: () -> (any SceneEntity)?) -> (any SceneEntity)? {
        // Check cache validity
        if Date().timeIntervalSince(lastUpdate) > maxAge {
            cache.removeAll()
            lastUpdate = Date()
        }
        
        // Check cache first
        if let cached = cache[point] {
            return cached
        }
        
        // Perform test and cache
        let result = test()
        cache[point] = result
        
        return result
    }
}
```

### Gesture Batching for Entities
```swift
class EntityTransformBatcher {
    private var pendingTransforms: [UUID: SIMD3<Float>] = [:]
    private var timer: Timer?
    
    func addTransform(for entity: any SceneEntity, delta: SIMD3<Float>) {
        pendingTransforms[entity.id, default: .zero] += delta
        
        // Batch updates
        timer?.invalidate()
        timer = Timer.scheduledTimer(withTimeInterval: 0.016, repeats: false) { _ in
            self.applyTransforms()
        }
    }
    
    private func applyTransforms() {
        for (entityId, totalDelta) in pendingTransforms {
            if let entity = sceneManager.entity(withId: entityId) {
                entity.position += totalDelta
            }
        }
        pendingTransforms.removeAll()
        viewportState.needsUpdate.toggle()
    }
}
```

## Entity-Specific Gesture Behaviors

### Camera Entity Gestures
```swift
// Double-tap to look through camera
.onTapGesture(count: 2) {
    if let camera = entity as? CameraEntity {
        viewportState.setActiveCamera(camera)
    }
}
```

### Light Entity Gestures
```swift
// Pinch to adjust light intensity
.gesture(
    MagnificationGesture()
        .onChanged { scale in
            if let light = selectedEntity as? LightEntity {
                light.intensity = baseIntensity * Float(scale)
            }
        }
)
```

### Model Entity Gestures
```swift
// Rotation gesture for models
.rotationEffect(rotationAngle)
.gesture(
    RotationGesture()
        .onChanged { angle in
            if let model = selectedEntity as? ModelEntity {
                model.rotation = simd_quatf(angle: Float(angle.radians), 
                                           axis: [0, 1, 0])
            }
        }
)
```

## Future Enhancements

- [ ] Custom gesture sets per entity type
- [ ] Gesture recording for entity animations
- [ ] Multi-entity selection gestures
- [ ] Advanced snapping during transform
- [ ] Apple Pencil precision mode
- [ ] Gesture-based entity creation
- [ ] VoiceOver gesture alternatives
- [ ] Game controller entity navigation

## Summary

The gesture system in v2.0:
- ✅ Fully integrated with Entity/ECS system
- ✅ Maps RealityKit.Entity hits to custom Entities
- ✅ Mode-based routing through ViewportState
- ✅ Platform-optimized while maintaining consistency
- ✅ Performance-optimized with caching and batching