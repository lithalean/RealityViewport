# RealityViewport Navigation Context

**Module**: NAVIGATION.md  
**Version**: 4.0  
**Architecture**: Simplified Entity Wrapper + Adaptive UI  
**Philosophy**: Navigate what's built, not what might be  
**Status**: Core Navigation Complete  
**Last Updated**: December 2024

## Navigation Philosophy

**"Simple flows for what exists, room for what's next."**

Navigation follows our core principle: ship what works today, add complexity when needed. Every flow documented here is **actually implemented**, not aspirational.

## Navigation State Machine

### Mode-Based Interaction
```
┌─────────────┐     [Space Key]    ┌──────────────┐
│ Environment │ ◄─────────────────► │    Entity    │
│    Mode     │                     │     Mode     │
└─────────────┘                     └──────────────┘
      │                                    │
      │ Camera Control                     │ Select Entity
      ▼                                    ▼
┌─────────────┐                     ┌──────────────┐
│   Orbit/    │                     │  Transform   │
│   Pan/Zoom  │                     │   Gizmos     │
└─────────────┘                     └──────────────┘
```

**What's Built:**
- ✅ Mode switching works
- ✅ Camera controls work
- ✅ Entity selection works
- ✅ Gizmos work (need polish)

**Not Built Yet (YAGNI):**
- ⏳ Complex tool modes
- ⏳ Advanced selection modes
- ⏳ Custom workspaces

## Screen Hierarchy (Adaptive UI)

```
RealityViewportApp
├── ContentView (SINGLE ADAPTIVE VIEW) ✅
│   ├── NavigationSplitView (iPad/Mac)
│   │   ├── Sidebar: InspectorView
│   │   └── Detail: ViewportStack
│   │
│   ├── NavigationStack (iPhone)
│   │   ├── Main: ViewportStack
│   │   └── Inspector: Sheet
│   │
│   └── ViewportStack (The Magic)
│       ├── MetalSkyView (GPU Sky) ✅
│       ├── ViewportMetalGrid (GPU Grid) ✅
│       └── ViewportView (RealityKit) ✅
│           └── Your entities render here
```

## Entity Type Disambiguation

### Navigation Always Knows Which Entity

```swift
// In navigation flows, be explicit:

// User selects in viewport
let hitEntity: RealityKit.Entity = raycast.entity  // RealityKit type

// Find wrapper
let wrapper: Entity? = sceneManager.findWrapper(for: hitEntity)

// Update selection
selectionManager.select(wrapper)  // Your Entity wrapper

// Create gizmo
viewportState.currentGizmo = createGizmo()  // RealityKit.Entity
```

## User Journey Flows

### 🎬 First Launch (What Actually Happens)
```
START → Empty Scene → See Sky + Grid
   ↓
Add Menu → Choose Entity Type
   ↓
Entity Created (simplified wrapper)
   ↓
See It → Select It → Transform It
   ↓
Save Project → Ship Your Game
```

**No Complex Onboarding** - You see the viewport, you start creating. Simple.

### 📁 Project Management (Working Today)
```
Projects Button → ProjectBrowserView
   ├─→ New: Creates empty scene ✅
   ├─→ Open: Loads .rvproject ✅
   ├─→ Save: Serializes entities ✅
   └─→ Recent: Quick access ✅

Not Built Yet:
   ⏳ Templates (YAGNI)
   ⏳ Cloud sync (YAGNI)
   ⏳ Version control (YAGNI)
```

### 💾 Import Model Flow (Simplified)
```
Import Button → Select USDZ/Reality
   ↓
ModelEntity created (wrapper)
   ↓
entity.load(from: url) // Async
   ↓
Appears at origin → Done

That's it. No complex import settings.
No material editors. It just works.
```

## Navigation Interactions

### The Bridge in Action

When navigating, remember the bridge pattern:

```swift
// User clicks in viewport
func handleViewportClick(at point: CGPoint) {
    // 1. Hit test returns RealityKit.Entity
    let rkEntity = hitTest(at: point)
    
    // 2. Find your wrapper
    let wrapper = sceneManager.findWrapper(for: rkEntity)
    
    // 3. Select wrapper
    selectionManager.select(wrapper)
    
    // 4. Gizmo uses RealityKit.Entity
    viewportState.currentGizmo = createGizmoEntity()
    viewportState.currentGizmo.position = wrapper.position
}
```

### Mode Switching

```swift
// Simple two-mode system
enum ViewportInteractionMode {
    case environment  // Camera
    case entity      // Selection
}

// Toggle with space
func toggleMode() {
    viewportState.interactionMode = 
        (viewportState.interactionMode == .environment) ? .entity : .environment
        
    // Update UI
    updateModeIndicator()
    updateCursor()
    updateGizmoVisibility()
}
```

## Platform Navigation Patterns

### Adaptive Layouts (Automatic)
```yaml
iPad/Mac (Regular Width):
  Layout: NavigationSplitView
  Inspector: Sidebar (320pt)
  Toolbar: Top overlay
  Gestures: Mouse/trackpad

iPhone (Compact Width):
  Layout: NavigationStack
  Inspector: Sheet overlay
  Toolbar: Bottom floating
  Gestures: Touch

Works automatically. No special code.
```

### Input Mapping

#### macOS (What's Built)
| Input | Mode | Result |
|-------|------|--------|
| Click + Drag | Environment | Orbit camera ✅ |
| Right Click + Drag | Any | Pan camera ✅ |
| Scroll | Any | Zoom ✅ |
| Click | Entity | Select ✅ |
| Space | Any | Toggle mode ✅ |

#### iOS (What's Built)
| Gesture | Mode | Result |
|---------|------|--------|
| One Finger Drag | Environment | Orbit ✅ |
| Two Finger Drag | Any | Pan ✅ |
| Pinch | Any | Zoom ✅ |
| Tap | Entity | Select ✅ |
| Long Press | Entity | Context menu ✅ |

## State Management

### Simple State, Clear Ownership

```swift
// SceneManager owns entities (your wrappers)
@Published var entities: [Entity] = []

// SelectionManager owns selection (your wrappers)
@Published var selectedEntity: Entity?

// ViewportState owns rendering (RealityKit.Entity)
let rootEntity = RealityKit.Entity()

// The bridge connects them
entity.realityEntity  // Links wrapper to rendering
```

### Navigation State
```yaml
Current Implementation:
  viewport_mode: .environment | .entity ✅
  camera_state: distance, azimuth, elevation ✅
  selection: single entity ✅
  gizmo: visible when selected ✅

Not Built (YAGNI):
  ⏳ Multi-selection UI
  ⏳ Selection groups
  ⏳ Navigation history
  ⏳ Saved views
```

## Navigation Flows

### Creating an Entity
```
1. User: Clicks Add → Light
2. System: Creates LightEntity (wrapper)
3. System: Adds to SceneManager
4. Bridge: wrapper.realityEntity → ViewportState
5. User: Sees light, can select it
```

### Selecting an Entity
```
1. User: Clicks in viewport
2. Hit Test: Returns RealityKit.Entity
3. Find Wrapper: SceneManager lookup
4. Update Selection: SelectionManager
5. Show Gizmo: At entity position
```

### Transforming an Entity
```
1. User: Drags gizmo axis
2. Gizmo: Updates position
3. Wrapper: entity.position updates
4. Bridge: Syncs to realityEntity
5. Render: ViewportState updates
```

## Error Handling

### Simple Recovery
```yaml
Model Load Fails:
  Show: "Couldn't load model"
  Action: Remove entity
  No complex recovery flows

Project Load Fails:
  Show: "Couldn't open project"
  Action: Keep current scene
  No data loss

That's it. Simple errors, simple recovery.
```

## Keyboard Shortcuts

### What's Actually Implemented
| Key | Action | Status |
|-----|--------|--------|
| Space | Toggle mode | ✅ Works |
| Delete | Delete entity | ✅ Works |
| Cmd+S | Save project | ✅ Works |
| Cmd+O | Open project | ✅ Works |

### Not Built Yet (YAGNI)
- Custom keybindings
- Shortcut editor
- Macro recording
- Command palette

## Navigation Performance

```yaml
Current Performance:
  Mode switch: Instant
  Selection: < 16ms
  Gizmo update: Real-time
  Scene load: < 1 second

No performance issues = No complex optimizations needed
```

## What's NOT in Navigation (And That's OK)

### Intentionally Simple
```yaml
Not Built:
  ❌ Node graph editor (use hierarchy)
  ❌ Timeline editor (no animation yet)
  ❌ Asset browser (just import)
  ❌ Layer system (not needed)
  ❌ Tool palettes (two modes enough)

Why: YAGNI - Add when games need them
```

### Future Navigation (If Needed)

```swift
// TODAY: Not needed
// FUTURE: Add if users request

// Example: Multi-select (when needed)
extension SelectionManager {
    var multiSelection: [Entity] = []  // Add later
}

// Example: Bookmarks (when needed)
extension ViewportState {
    var savedViews: [CameraState] = []  // Add later
}
```

## Navigation Best Practices

### DO: Keep It Simple
```swift
// Good - Direct navigation
func selectEntity(_ entity: Entity) {
    selectionManager.selectedEntity = entity
}

// Over-engineered - Don't do this yet
func selectEntity(_ entity: Entity, 
                  mode: SelectionMode,
                  options: SelectionOptions,
                  animation: AnimationStyle) { }
```

### DO: Use The Bridge
```swift
// Wrapper for UI/logic
let entity = Entity()

// Bridge for rendering
viewportState.rootEntity.addChild(entity.realityEntity)

// Never mix them in navigation
```

### DON'T: Build Navigation for Nonexistent Features
```swift
// Don't build navigation for:
- Features you haven't built
- Workflows users haven't requested
- Optimizations you don't need
```

## Platform-Specific Notes

### macOS
- Menu bar works ✅
- Keyboard shortcuts work ✅
- No touch bar (deprecated)
- No complex panels (YAGNI)

### iOS/iPadOS
- Touch works ✅
- Gestures work ✅
- Haptics work ✅
- No pencil-specific features (YAGNI)

### tvOS
- Basic navigation ✅
- Limited testing ⚠️
- Simplified features
- Remote control works

## See Also
- **Architecture.md** - Overall navigation philosophy
- **EntitySystem.md** - What entities can do
- **ViewportState.md** - How rendering connects
- **Gestures.md** - Input handling details

## Summary

Navigation v4.0 is **intentionally simple**:
- ✅ **Two modes** - Environment and Entity (enough for now)
- ✅ **Clear flows** - Create, select, transform, save
- ✅ **Adaptive UI** - Works on all platforms automatically
- ✅ **Entity-based** - Your wrappers for logic, RealityKit for rendering
- ✅ **Room to grow** - Add navigation as features emerge

**Philosophy**: Navigate what exists. Don't over-engineer. Ship today.