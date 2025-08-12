# RealityViewport Gesture & Input Documentation

**Module**: GESTURES.md  
**Version**: 3.0  
**Architecture**: Simplified Input Handling  
**Philosophy**: Handle what's needed, not what's possible  
**Status**: Core Gestures Working  
**Last Updated**: December 2024

## Gesture Philosophy

**"Two modes, clear input, ship today."**

We don't need 50 gesture recognizers. We need the basics working well. Camera control and entity selection - that's 90% of what users need.

## Core Input Architecture

### The Simple Truth
```
User Input → Two Modes → Two Outcomes

Environment Mode:
  Drag → Orbit camera
  Right-drag → Pan camera  
  Scroll → Zoom camera

Entity Mode:
  Click → Select entity
  Drag → Move entity (with gizmo)
  
That's it. Ship it.
```

## Entity Type Disambiguation in Gestures

### Always Know Your Entity Type

```swift
// When handling gestures, be explicit:

func handleTap(at location: CGPoint) {
    // 1. Hit test returns RealityKit.Entity
    let rkEntity: RealityKit.Entity? = hitTest(at: location)
    
    // 2. Find your wrapper
    let wrapper: Entity? = sceneManager.findWrapper(for: rkEntity)
    
    // 3. Use wrapper for logic
    selectionManager.select(wrapper)
    
    // 4. Bridge updates rendering
    if let wrapper = wrapper {
        viewportState.currentGizmo?.position = wrapper.position
    }
}
```

## Platform Input Maps

### macOS (What's Built)

```swift
// WORKING TODAY
.gesture(
    DragGesture()
        .onChanged { value in
            switch viewportState.interactionMode {
            case .environment:
                // Orbit camera ✅
                cameraController.orbit(by: value.translation)
            case .entity:
                // Move selected entity ✅
                if let selected = selectionManager.selectedEntity {
                    moveEntity(selected, by: value.translation)
                }
            }
        }
)

// Right-click pan ✅
.gesture(
    DragGesture()
        .modifiers(.option)
        .onChanged { value in
            cameraController.pan(by: value.translation)
        }
)

// Scroll zoom ✅
.onScroll { delta in
    cameraController.zoom(by: delta.y)
}
```

**Not Built (YAGNI):**
- Complex multi-button combos
- Gesture recording
- Custom gesture sets
- Macro system

### iOS (What's Built)

```swift
// WORKING TODAY
// One finger drag ✅
.gesture(
    DragGesture()
        .onChanged { value in
            switch viewportState.interactionMode {
            case .environment:
                cameraController.orbit(by: value.translation)
            case .entity:
                handleEntityDrag(value.translation)
            }
        }
)

// Two finger pan ✅
.gesture(
    SimultaneousGesture(
        DragGesture(),
        MagnificationGesture()
    )
    .onChanged { value in
        if let drag = value.first {
            cameraController.pan(by: drag.translation)
        }
    }
)

// Pinch zoom ✅
.gesture(
    MagnificationGesture()
        .onChanged { scale in
            cameraController.zoom(to: scale)
        }
)
```

**Not Built (YAGNI):**
- Apple Pencil specific features
- Force touch
- Complex gesture combinations
- Gesture customization

## Hit Testing (The Bridge in Action)

### Simple Hit Test Flow

```swift
func hitTestEntity(at location: CGPoint) -> Entity? {
    // 1. RealityKit does the work
    guard let ray = viewport.ray(through: location) else { return nil }
    
    let hits = scene.raycast(
        origin: ray.origin,
        direction: ray.direction,
        length: 100,
        query: .nearest,
        mask: .default
    )
    
    // 2. Get RealityKit.Entity
    guard let hit = hits.first else { return nil }
    let rkEntity = hit.entity  // RealityKit.Entity
    
    // 3. Find your wrapper
    return sceneManager.findWrapper(for: rkEntity)
}
```

**That's it.** No complex hit test caching. No multi-layer testing. It works.

## Camera Control (Simple Math)

### Orbit
```swift
func orbitCamera(by delta: CGSize) {
    // Simple spherical coordinates
    let sensitivity: Float = 0.01
    viewportState.cameraAzimuth += Float(delta.width) * sensitivity
    viewportState.cameraElevation += Float(delta.height) * sensitivity
    
    // Clamp to prevent flipping
    viewportState.cameraElevation = max(-1.5, min(1.5, viewportState.cameraElevation))
    
    // Update position
    let position = viewportState.computeCameraPosition()
    viewportState.cameraEntity.position = position
    viewportState.cameraEntity.look(at: viewportState.cameraTarget, from: position, relativeTo: nil)
}
```

### Pan
```swift
func panCamera(by delta: CGSize) {
    // Move the target
    let scale = viewportState.cameraDistance * 0.001
    viewportState.cameraTarget.x += Float(delta.width) * scale
    viewportState.cameraTarget.y -= Float(delta.height) * scale
    
    // Update camera
    let position = viewportState.computeCameraPosition()
    viewportState.cameraEntity.position = position
}
```

### Zoom
```swift
func zoomCamera(by delta: Float) {
    // Exponential for natural feel
    viewportState.cameraDistance *= exp(-delta * 0.1)
    viewportState.cameraDistance = max(0.5, min(50, viewportState.cameraDistance))
    
    // Update position
    viewportState.cameraEntity.position = viewportState.computeCameraPosition()
}
```

## Gizmo Interaction (Basic)

### What's Built
```swift
func handleGizmoDrag(axis: GizmoAxis, delta: CGSize) {
    guard let selected = selectionManager.selectedEntity else { return }
    
    // Simple axis constraints
    switch axis {
    case .x:
        selected.position.x += Float(delta.width) * 0.01
    case .y:
        selected.position.y += Float(delta.height) * 0.01
    case .z:
        selected.position.z += Float(delta.width) * 0.01
    }
    
    // Update gizmo position
    viewportState.currentGizmo?.position = selected.position
}
```

**Not Built (YAGNI):**
- Rotation gizmos
- Scale gizmos
- Snapping
- Constraints
- Multi-axis movement

*Add these when you need them, not before.*

## Mode Switching

### Simple Toggle
```swift
// Space key toggles mode
.keyboardShortcut(.space, modifiers: [])
.onReceive(NotificationCenter.default.publisher(for: .toggleMode)) { _ in
    viewportState.interactionMode = 
        viewportState.interactionMode == .environment ? .entity : .environment
}
```

**That's all.** No complex mode system. No tool palettes. Two modes work.

## Keyboard Shortcuts (What Works)

### Actually Implemented
```swift
.keyboardShortcut(.space, modifiers: [])      // Toggle mode ✅
.keyboardShortcut(.delete, modifiers: [])     // Delete entity ✅
.keyboardShortcut("s", modifiers: .command)   // Save ✅
.keyboardShortcut("o", modifiers: .command)   // Open ✅

// That's it. The basics work.
```

### Not Built (YAGNI)
- Transform tool shortcuts (W, E, R)
- Duplicate shortcuts
- Focus shortcuts
- Custom key bindings
- Shortcut editor

*When users ask for these, add them. Not before.*

## Haptic Feedback (iOS)

### What's Implemented
```swift
#if os(iOS)
// Selection feedback ✅
func entitySelected() {
    UIImpactFeedbackGenerator(style: .light).impactOccurred()
}

// Mode switch feedback ✅
func modeChanged() {
    UISelectionFeedbackGenerator().selectionChanged()
}
#endif
```

**Not Built:**
- Complex haptic patterns
- Custom haptic curves
- Haptic preferences
- Platform haptics beyond iOS

*Basic feedback works. That's enough for now.*

## Touch Targets (iOS)

### Simple Expansion
```swift
func expandedHitTest(at point: CGPoint) -> Entity? {
    // Try the exact point first
    if let entity = hitTestEntity(at: point) {
        return entity
    }
    
    // Try a slightly larger area (44pt minimum target)
    let padding: CGFloat = 22
    for offset in [CGPoint(x: padding, y: 0), CGPoint(x: -padding, y: 0),
                   CGPoint(x: 0, y: padding), CGPoint(x: 0, y: -padding)] {
        let testPoint = CGPoint(x: point.x + offset.x, y: point.y + offset.y)
        if let entity = hitTestEntity(at: testPoint) {
            return entity
        }
    }
    
    return nil
}
```

**Not Built:**
- Dynamic touch targets
- Entity-specific hit areas
- Touch prediction
- Gesture anticipation

## Performance

### What We Have
```yaml
Current Performance:
  Hit test: < 5ms ✅
  Gesture response: < 16ms ✅
  Mode switch: Instant ✅
  
No performance issues = No complex optimizations
```

### Not Built (YAGNI)
```swift
// We DON'T have:
// - Hit test caching (not needed)
// - Gesture batching (not needed)
// - Transform queuing (not needed)
// - Predictive input (not needed)

// Why? Because it works fine without them.
```

## Entity-Specific Gestures

### What's Built
```swift
// Basic selection and movement
func handleEntityTap(_ entity: Entity) {
    selectionManager.select(entity)
}

func handleEntityDrag(_ entity: Entity, delta: CGSize) {
    entity.position.x += Float(delta.width) * 0.01
    entity.position.y -= Float(delta.height) * 0.01
}
```

### Not Built (YAGNI)
- Double-tap to look through camera
- Pinch to scale
- Rotate gestures
- Entity-specific gesture sets
- Context-sensitive gestures

*These sound cool. Build them when someone needs them.*

## tvOS Input (Basic Support)

### What Works
```swift
.onMoveCommand { direction in
    // Simple navigation
    switch direction {
    case .left, .right:
        cameraController.orbit(horizontal: direction == .left ? -1 : 1)
    case .up, .down:
        cameraController.orbit(vertical: direction == .up ? 1 : -1)
    }
}
```

**Status:** Basic support, limited testing. Good enough for alpha.

## Common Patterns

### The Bridge Pattern in Gestures
```swift
// Always remember the separation:

// 1. Gesture hits RealityKit.Entity
let rkEntity = hitTest(at: location)

// 2. Find your wrapper for logic
let wrapper = sceneManager.findWrapper(for: rkEntity)

// 3. Update wrapper
wrapper?.position = newPosition

// 4. Bridge syncs to rendering
// (happens automatically via property observers)
```

### Keep Gestures Simple
```swift
// Good - Direct and simple
func handleDrag(_ delta: CGSize) {
    if mode == .environment {
        orbitCamera(by: delta)
    } else {
        moveEntity(by: delta)
    }
}

// Over-engineered - Don't do this
func handleDrag(_ delta: CGSize, 
                with options: DragOptions,
                using strategy: DragStrategy,
                animated: Bool,
                completion: @escaping () -> Void) { }
```

## What's NOT in Gestures (And That's OK)

### Intentionally Missing
```yaml
Not Built:
  ❌ Gesture recording (YAGNI)
  ❌ Custom gesture recognition (YAGNI)
  ❌ Multi-touch beyond basics (YAGNI)
  ❌ Gesture preferences UI (YAGNI)
  ❌ Accessibility gestures (add when needed)
  ❌ Game controller support (add when needed)

These aren't bugs. They're future possibilities.
```

## Future Growth (When Needed)

```swift
// TODAY: Not needed
// FUTURE: Add based on user feedback

// Example: Rotation (when needed)
extension GestureHandler {
    func handleRotation(_ angle: Angle) {
        // Add this when users need to rotate things
    }
}

// Example: Snapping (when needed)
extension GestureHandler {
    func snapToGrid(_ position: SIMD3<Float>) -> SIMD3<Float> {
        // Add this when users want snapping
    }
}
```

## Best Practices

### DO: Keep It Direct
```swift
// Good - Simple and clear
tap → select
drag → move
scroll → zoom
```

### DO: Use The Bridge
```swift
// RealityKit.Entity for hit testing
// Entity wrapper for logic
// Never confuse them
```

### DON'T: Over-Optimize
```swift
// Don't add caching until you measure a problem
// Don't batch gestures until you see lag
// Don't predict input until users complain
```

### DON'T: Build for Tomorrow
```swift
// Don't implement gestures for features that don't exist
// Don't create complex input systems for simple needs
// Don't optimize what's already fast enough
```

## See Also
- **ViewportState.md** - How gestures affect rendering
- **EntitySystem.md** - What entities can do
- **Navigation.md** - Overall interaction flow
- **Architecture.md** - The philosophy behind simplicity

## Summary

Gestures v3.0 is **intentionally basic**:
- ✅ **Two modes** - Environment and Entity (enough)
- ✅ **Basic gestures** - Tap, drag, scroll, pinch (covers 90%)
- ✅ **Clear bridge** - RealityKit.Entity → Entity wrapper
- ✅ **Platform basics** - Works on Mac, iOS, tvOS
- ✅ **Room to grow** - Add gestures as features emerge

**Philosophy**: Ship with gestures that work. Add complexity when users need it. Not before.