# RealityViewport Gesture & Input Documentation

**Purpose**: Document sophisticated gesture and input handling  
**Version**: 1.0  
**Complexity**: Platform-specific with unified behavior  
**Last Updated**: July 2025

## Overview

RealityViewport implements a mode-based gesture system that adapts to each platform while maintaining consistent behavior. Input is routed through ViewportState modes to appropriate handlers.

## Mode-Based Gesture Routing

### Input Flow
```
User Input → Platform Gesture → ViewportState.currentMode → Handler
                                        ↓
                              Environment Mode → CameraController
                                        ↓
                                Entity Mode → SelectionManager/Gizmo
```

### Mode Definitions
```swift
enum ViewportInteractionMode {
    case environment  // Camera manipulation
    case entity      // Object selection/transformation
    
    func handleDrag(_ translation: CGSize, in viewport: ViewportView) {
        switch self {
        case .environment:
            viewport.cameraController.handleOrbitDrag(translation)
        case .entity:
            viewport.handleEntityDrag(translation)
        }
    }
}
```

## Platform-Specific Input Differences

### macOS Input Mapping
```swift
// Primary Input
.gesture(
    DragGesture(minimumDistance: 1)
        .onChanged { value in
            if viewportState.currentMode == .environment {
                // Left drag = orbit
                cameraController.orbit(by: value.translation)
            } else {
                // Left drag = select/move
                handleEntityInteraction(at: value.location)
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

### iOS Input Mapping
```swift
// Single Touch
.gesture(
    DragGesture(minimumDistance: 10)
        .onChanged { value in
            handleModeBasedDrag(value.translation)
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

// Long Press for Context
.gesture(
    LongPressGesture(minimumDuration: 0.5)
        .onEnded { _ in
            showContextMenu()
        }
)
```

### tvOS Input Mapping
```swift
// Siri Remote Swipes
.onMoveCommand { direction in
    switch direction {
    case .left, .right:
        cameraController.orbit(horizontal: direction == .left ? -1 : 1)
    case .up, .down:
        cameraController.orbit(vertical: direction == .up ? 1 : -1)
    }
}

// Play/Pause Button
.onPlayPauseCommand {
    viewportState.toggleInteractionMode()
}

// Menu Button
.onExitCommand {
    showMainMenu()
}
```

## Gizmo Hit Detection

### Hit Test Implementation
```swift
func hitTestGizmo(at location: CGPoint) -> GizmoAxis? {
    // Convert screen to world coordinates
    let ray = viewport.ray(from: location)
    
    // Test against gizmo components
    if let hit = scene.raycast(
        origin: ray.origin,
        direction: ray.direction,
        length: 100,
        query: .nearest,
        mask: .gizmo
    ).first {
        return gizmoAxis(for: hit.entity)
    }
    
    return nil
}
```

### Gizmo Interaction States
```swift
enum GizmoInteractionState {
    case idle
    case hovering(axis: GizmoAxis)
    case dragging(axis: GizmoAxis, startPoint: SIMD3<Float>)
    
    var cursorStyle: NSCursor? {
        switch self {
        case .idle:
            return nil
        case .hovering(let axis):
            return axis.cursor
        case .dragging:
            return .closedHand
        }
    }
}
```

### Axis-Constrained Movement
```swift
func dragGizmo(axis: GizmoAxis, by delta: CGSize) {
    guard let selected = viewportState.selectedEntity else { return }
    
    // Convert screen delta to world space
    let worldDelta = screenToWorld(delta)
    
    // Apply constraint
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
    // ... other combinations
    }
    
    selected.position += movement
    viewportState.triggerUpdate()
}
```

## Camera Control Mathematics

### Orbit Calculation
```swift
func orbitCamera(by delta: CGSize) {
    // Convert pixel movement to radians
    let sensitivity: Float = 0.01
    let deltaX = Float(delta.width) * sensitivity
    let deltaY = Float(delta.height) * sensitivity
    
    // Update spherical coordinates
    viewportState.cameraRotation.x += deltaX
    viewportState.cameraRotation.y += deltaY
    
    // Clamp elevation to prevent flipping
    viewportState.cameraRotation.y = viewportState.cameraRotation.y
        .clamped(to: -.pi/2 + 0.1...pi/2 - 0.1)
    
    // Update camera position
    updateCameraTransform()
}
```

### Pan Calculation
```swift
func panCamera(by delta: CGSize) {
    // Get camera right and up vectors
    let transform = cameraEntity.transform
    let right = transform.matrix.columns.0.xyz
    let up = transform.matrix.columns.1.xyz
    
    // Scale by distance for consistent movement
    let scale = viewportState.cameraDistance * 0.001
    let deltaX = Float(delta.width) * scale
    let deltaY = Float(delta.height) * scale
    
    // Move target
    viewportState.cameraTarget += right * deltaX
    viewportState.cameraTarget += up * -deltaY
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
}
```

## Touch Target Optimization

### iOS Touch Areas
```swift
struct TouchTargets {
    static let minimumTapTarget: CGFloat = 44  // Apple HIG
    static let gizmoHitPadding: CGFloat = 20   // Extra hit area
    static let entitySelectPadding: CGFloat = 10
}

func expandedHitTest(at point: CGPoint) -> Entity? {
    // Test expanded area for better touch accuracy
    let testPoints = [
        point,
        point.offset(dx: TouchTargets.entitySelectPadding, dy: 0),
        point.offset(dx: -TouchTargets.entitySelectPadding, dy: 0),
        point.offset(dx: 0, dy: TouchTargets.entitySelectPadding),
        point.offset(dx: 0, dy: -TouchTargets.entitySelectPadding)
    ]
    
    for testPoint in testPoints {
        if let hit = viewport.hitTest(at: testPoint) {
            return hit
        }
    }
    
    return nil
}
```

## Gesture Conflict Resolution

### Simultaneous Gesture Handling
```swift
// Allow pan and zoom together
panGesture.simultaneously(with: pinchGesture)

// Exclusive mode switching
tapGesture.exclusively(before: dragGesture)
```

### Priority System
```swift
enum GesturePriority: Int {
    case modeToggle = 1000      // Highest
    case gizmoInteraction = 900
    case entitySelection = 800
    case cameraControl = 700    // Lowest
}
```

## Haptic Feedback (iOS)

### Feedback Patterns
```swift
enum HapticPattern {
    case selection
    case modeSwitch
    case gizmoSnap
    case error
    
    func trigger() {
        #if os(iOS)
        switch self {
        case .selection:
            UIImpactFeedbackGenerator(style: .light).impactOccurred()
        case .modeSwitch:
            UIImpactFeedbackGenerator(style: .medium).impactOccurred()
        case .gizmoSnap:
            UINotificationFeedbackGenerator().notificationOccurred(.success)
        case .error:
            UINotificationFeedbackGenerator().notificationOccurred(.error)
        }
        #endif
    }
}
```

## Keyboard Shortcuts (macOS)

### Shortcut Definitions
```swift
extension View {
    func viewportKeyboardShortcuts() -> some View {
        self
            .keyboardShortcut("w", modifiers: [])  // Move tool
            .keyboardShortcut("e", modifiers: [])  // Rotate tool
            .keyboardShortcut("r", modifiers: [])  // Scale tool
            .keyboardShortcut(" ", modifiers: [])  // Toggle mode
            .keyboardShortcut("f", modifiers: [])  // Focus selected
            .keyboardShortcut("g", modifiers: [])  // Toggle grid
            .keyboardShortcut("delete", modifiers: [])  // Delete
    }
}
```

## Performance Optimizations

### Gesture Debouncing
```swift
class GestureDebouncer {
    private var workItem: DispatchWorkItem?
    
    func debounce(delay: TimeInterval = 0.1, action: @escaping () -> Void) {
        workItem?.cancel()
        workItem = DispatchWorkItem(block: action)
        DispatchQueue.main.asyncAfter(
            deadline: .now() + delay,
            execute: workItem!
        )
    }
}
```

### Hit Test Caching
```swift
struct HitTestCache {
    private var cache: [CGPoint: Entity?] = [:]
    private let maxAge: TimeInterval = 0.1
    
    mutating func hitTest(at point: CGPoint, 
                         using test: () -> Entity?) -> Entity? {
        // Check cache first
        if let cached = cache[point] {
            return cached
        }
        
        // Perform test and cache
        let result = test()
        cache[point] = result
        
        // Clear old entries periodically
        if cache.count > 100 {
            cache.removeAll()
        }
        
        return result
    }
}
```

## Future Enhancements

- [ ] Gesture customization preferences
- [ ] Advanced multi-touch gestures
- [ ] Gesture recording/playback
- [ ] Accessibility gesture alternatives
- [ ] Force touch support (where available)
- [ ] Apple Pencil integration
- [ ] Game controller support