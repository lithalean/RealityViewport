# RealityViewport ViewportState Documentation

**Module**: VIEWPORTSTATE.md  
**Version**: 4.0  
**Architecture**: RealityKit.Entity Bridge  
**Philosophy**: Clean separation between wrapper and rendering  
**Status**: Core System - 95% Complete  
**Last Updated**: August 25 2025

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

## Platform-Specific Update Strategies (v4.0)

### The iOS Publishing Fix
```yaml
Problem (iOS):
  - RealityView update closure modifying @Published
  - "Publishing changes from within view updates" error
  - Caused app to become unusable
  
Solution:
  iOS: Timer-based updates at 60fps
  macOS: On-demand updates with needsUpdate flag
  
Result:
  - Smooth 60fps on all platforms
  - No SwiftUI conflicts
  - Consistent performance
```

### Implementation Pattern
```swift
// ViewportView.swift - Platform-specific updates
struct ViewportView: View {
    #if os(iOS)
    @State private var updateTimer: Timer?
    #endif
    
    var body: some View {
        RealityView { content in
            // Initial setup
        } update: { content in
            #if !os(iOS)
            // macOS: On-demand updates
            if viewportState.needsUpdate {
                updateEntities()
                updateCamera()
                updateGizmo()
                viewportState.needsUpdate = false
            }
            #endif
        }
        #if os(iOS)
        .onAppear {
            // iOS: Timer-based updates
            updateTimer = Timer.scheduledTimer(withTimeInterval: 1.0/60.0, repeats: true) { _ in
                Task { @MainActor in
                    updateEntities()
                    updateCamera()
                    updateGizmo()
                    BillboardSystem.update(
                        in: viewportState.rootEntity,
                        cameraTransform: viewportState.cameraEntity.transform
                    )
                }
            }
        }
        .onDisappear {
            updateTimer?.invalidate()
            updateTimer = nil
        }
        #endif
    }
}
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
  Updates: Platform-optimized (timer vs on-demand)
  
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
    var mouseLocation: CGPoint = .zero          // Mouse position (macOS)
    var lastDragValue: CGSize = .zero           // Drag tracking
    @Published var needsUpdate: Bool = false    // Update trigger (macOS primarily)
    
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
  - Platform-Optimized: Different update strategies
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
  Range: 1.0 to 50.0 (clamped)
  
cameraAzimuth:
  Type: Float (radians)
  Default: 0.785 (45°)
  Purpose: Horizontal orbit angle
  Updates: Continuous rotation
  
cameraElevation:
  Type: Float (radians)
  Default: 0.523 (30°)
  Purpose: Vertical orbit angle
  Range: -π/2+0.1 to π/2-0.1 (avoid gimbal lock)
  
cameraTarget:
  Type: SIMD3<Float>
  Default: [0, 0, 0]
  Purpose: Point camera orbits around
  Updates: Pan gestures move this
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
    // Clear selection
    sceneManager.clearSelection()
case .entity:
    // Drag moves selected entity (via gizmo)
    if let selected = selectionManager.selectedEntity {
        handleGizmoInteraction(dragDelta)
    }
}
```

## State Management

### Minimal Published Properties
```swift
// Only 2 @Published for performance
@Published var needsUpdate: Bool = false        // Primarily for macOS
@Published var interactionMode: ViewportInteractionMode = .environment
```

### Platform-Specific Update Patterns
```swift
// macOS: Efficient on-demand updates
func triggerViewportUpdate() {
    #if os(macOS)
    viewportState.needsUpdate = true  // Triggers RealityView update
    #endif
}

// iOS: Timer handles updates automatically
// No need to set needsUpdate on iOS
```

## Update Strategy Comparison

### macOS Strategy (On-Demand)
```yaml
Trigger: needsUpdate flag
Benefits:
  - Battery efficient
  - Updates only when needed
  - Lower CPU usage
Pattern:
  - User interaction sets needsUpdate
  - RealityView update closure checks flag
  - Resets flag after update
```

### iOS Strategy (Timer-Based)
```yaml
Trigger: 60fps timer
Benefits:
  - Avoids SwiftUI publishing conflicts
  - Smooth continuous updates
  - Consistent frame rate
Pattern:
  - Timer fires every 16.67ms
  - Updates wrapped in Task { @MainActor }
  - No needsUpdate flag required
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
    
    // Scale based on camera distance
    let gizmoScale = cameraDistance * 0.1
    gizmo.scale = SIMD3<Float>(repeating: gizmoScale)
    
    rootEntity.addChild(gizmo)
    currentGizmo = gizmo
}

// Hide gizmo
func hideGizmo() {
    currentGizmo?.removeFromParent()
    currentGizmo = nil
}
```

## Integration with Floating UI

### No Direct UI Knowledge
```yaml
ViewportState:
  - Knows nothing about floating panels
  - Doesn't track inspector state
  - Doesn't manage toolbar
  - Pure rendering focus

UI State (ContentView):
  - @State showInspector: Bool
  - Floating panel positions
  - Glass morphism effects
  - All UI in ZStack above viewport
```

### Clean Separation
```swift
// ViewportState provides the 3D view
struct ViewportView: View {
    @ObservedObject var viewportState: ViewportState
    
    var body: some View {
        RealityView { content in
            content.add(viewportState.rootEntity)
        }
        .background(Color.clear)  // Transparent for Metal layers
    }
}

// ContentView adds floating UI
struct ContentView: View {
    var body: some View {
        ZStack {
            ViewportView()  // Full screen
            FloatingToolbar()  // Overlaid
            FloatingInspector()  // Overlaid
        }
    }
}
```

## Performance Considerations

### Platform-Optimized Performance
```yaml
macOS Performance:
  - On-demand updates only
  - ~11-15ms frame time
  - Battery efficient
  - Minimal CPU usage when idle

iOS Performance:
  - Consistent 60fps via timer
  - ~11-15ms frame time
  - Avoids publishing conflicts
  - Smooth interaction

Shared Optimizations:
  - Direct RealityKit.Entity manipulation
  - Minimal @Published properties
  - No wrapper overhead
  - Efficient bridge pattern
```

## Common Mistakes to Avoid

### ❌ DON'T: Mix Entity Types
```swift
// WRONG - Type mismatch
viewportState.rootEntity = Entity()  // Can't assign wrapper to RealityKit.Entity

// CORRECT - Use the bridge
viewportState.rootEntity.addChild(entity.realityEntity)
```

### ❌ DON'T: Forget Platform Differences
```swift
// WRONG - Using needsUpdate on iOS
#if os(iOS)
viewportState.needsUpdate = true  // Won't trigger updates on iOS
#endif

// CORRECT - Platform-specific patterns
#if os(macOS)
viewportState.needsUpdate = true  // macOS on-demand
#else
// iOS uses timer, no manual trigger needed
#endif
```

### ❌ DON'T: Update in Wrong Context
```swift
// WRONG - Direct updates in RealityView update closure on iOS
update: { _ in
    viewportState.somePublishedProperty = newValue  // Publishing error!
}

// CORRECT - Defer with Task on iOS
update: { _ in
    Task { @MainActor in
        // Safe to update here
    }
}
```

## Future Enhancements

### Planned (When Needed)
- [ ] Multiple viewport support (split views)
- [ ] Camera bookmarks (saved positions)
- [ ] Performance metrics overlay
- [ ] Unified update strategy (when iOS allows)

### Maybe Never (YAGNI)
- [ ] Complex viewport layouts
- [ ] Multiple camera types
- [ ] Advanced render settings
- [ ] Post-processing pipeline

## Recent Improvements (v4.0)

### August 2025 Session
```yaml
Fixed:
  ✅ iOS "Publishing changes" error
  ✅ Viewport becoming unusable on iPhone
  ✅ Inconsistent update behavior

Implemented:
  ✅ Platform-specific update strategies
  ✅ Timer-based iOS updates
  ✅ Clean Task { @MainActor } pattern
  ✅ Maintained 60fps on all platforms

Result:
  - Stable performance
  - No SwiftUI conflicts
  - Clean platform separation
```

## See Also
- **Implementation.md** - iOS performance fix details
- **EntitySystem.md** - Your simplified Entity wrapper
- **Architecture.md** - Bridge pattern explanation
- **ViewportView.swift** - Actual implementation

## Summary

ViewportState v4.0 is the **platform-optimized rendering bridge** that:
- ✅ **Uses RealityKit.Entity directly** - No wrapper overhead
- ✅ **Platform-specific updates** - Timer (iOS) vs on-demand (macOS)
- ✅ **Avoids SwiftUI conflicts** - Task { @MainActor } pattern
- ✅ **Bridges via .realityEntity** - Clean connection point
- ✅ **Maintains 60fps** - Smooth on all platforms
- ✅ **Stays focused** - Just viewport management, no UI knowledge

The platform-specific update strategies ensure smooth performance while avoiding SwiftUI publishing conflicts, especially on iOS where timer-based updates proved essential.