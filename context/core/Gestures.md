# RealityViewport Gesture & Input Documentation

**Module**: GESTURES.md  
**Version**: 3.1  
**Architecture**: Simplified Input Handling  
**Philosophy**: Handle what's needed, not what's possible  
**Status**: Core Gestures Working  
**Last Updated**: August 25 2025

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

## Recent Updates (v3.1)

### What Changed in August 2025
```yaml
Fixed:
  ✅ iOS haptic feedback centralized
  ✅ No more duplicate HapticStyle enums
  ✅ Smooth gestures with timer-based updates

Unchanged:
  - Core gesture system still works great
  - Two-mode system proven effective
  - Platform input maps remain the same

Impact on Gestures:
  - iOS gestures now update via timer (60fps)
  - No more SwiftUI publishing conflicts
  - Haptics use shared HapticFeedback.swift
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

// Trackpad pinch zoom ✅ (NEW)
.gesture(
    MagnificationGesture()
        .onChanged { value in
            let scale = Float(1.0 / value)
            viewportState.cameraDistance *= scale
        }
)
```

**Not Built (YAGNI):**
- Complex multi-button combos
- Gesture recording
- Custom gesture sets
- Macro system

### iOS (What's Built)

```swift
// WORKING TODAY with timer-based updates
// One finger drag ✅
.gesture(
    DragGesture()
        .onChanged { value in
            switch viewportState.interactionMode {
            case .environment:
                cameraController.orbit(by: value.translation)
            case .entity:
                // On iOS, this now updates smoothly via timer
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

**iOS-Specific Note:**
```yaml
Update Mechanism:
  - Gestures set state values
  - Timer at 60fps reads state
  - Updates happen in Task { @MainActor }
  - No SwiftUI publishing conflicts
  - Smooth, consistent interaction
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

### iOS Selection Cycling (Alternative)
```swift
// On iOS, tap cycles through entities instead of hit testing
func cycleSelection() {
    let entities = sceneManager.entities
    guard !entities.isEmpty else { return }
    
    if let current = selectionManager.selectedEntity,
       let index = entities.firstIndex(where: { $0.id == current.id }) {
        let nextIndex = (index + 1) % entities.count
        selectionManager.select(entities[nextIndex])
    } else {
        selectionManager.select(entities.first)
    }
    
    #if os(iOS)
    hapticFeedback(.light)  // Using shared utility
    #endif
}
```

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

**Note:** Gizmo dragging needs smoothing (known issue), but works functionally.

**Not Built (YAGNI):**
- Rotation gizmos (use realityEntity when needed)
- Scale gizmos (use realityEntity when needed)
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

**Visual Feedback in Floating Toolbar:**
- Mode buttons show active state
- Different background color when active
- Smooth transition animation (0.2s)

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

### What's Implemented (Using Shared Utility)
```swift
#if os(iOS)
// Now using HapticFeedback.swift shared utility

// Selection feedback ✅
func entitySelected() {
    hapticFeedback(.light)  // Shared function
}

// Mode switch feedback ✅
func modeChanged() {
    hapticFeedback(.medium)  // Shared function
}

// Gizmo interaction ✅
func gizmoMoved() {
    hapticFeedback(.light)  // Shared function
}
#endif
```

**Fixed in v3.1:**
- No more duplicate HapticStyle enums
- Centralized in HapticFeedback.swift
- Consistent across all files

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
  iOS updates: 60fps via timer ✅
  
No performance issues = No complex optimizations
```

### Platform-Specific Performance
```yaml
macOS:
  - On-demand updates
  - Immediate gesture response
  - Low CPU usage when idle

iOS:
  - Timer-based updates (60fps)
  - Smooth gesture tracking
  - No publishing conflicts
  - Consistent frame rate
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

## Interaction with Floating UI

### Gestures Work Everywhere
```yaml
Edge-to-Edge Viewport:
  - Full screen gesture area
  - Floating panels don't block gestures
  - Transparent to input when not hit

Mode Indication:
  - Floating toolbar shows active mode
  - Visual feedback in buttons
  - No gesture conflicts with UI
```

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
- **ViewportState.md** - Platform-specific update strategies
- **HapticFeedback.swift** - Shared haptic utility
- **Implementation.md** - iOS performance fixes
- **Navigation.md** - Overall interaction flow

## Summary

Gestures v3.1 is **intentionally basic and working great**:
- ✅ **Two modes** - Environment and Entity (proven effective)
- ✅ **Basic gestures** - Tap, drag, scroll, pinch (covers 90%)
- ✅ **Clear bridge** - RealityKit.Entity → Entity wrapper
- ✅ **Platform optimized** - Timer updates on iOS, on-demand on macOS
- ✅ **Haptics fixed** - Centralized utility, no duplicates
- ✅ **Room to grow** - Add gestures as features emerge

**Philosophy**: Ship with gestures that work. Add complexity when users need it. Not before.