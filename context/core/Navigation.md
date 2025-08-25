# RealityViewport Navigation Context

**Module**: NAVIGATION.md  
**Version**: 5.0  
**Architecture**: Simplified Entity Wrapper + Floating UI  
**Philosophy**: Navigate what's built, not what might be  
**Status**: Core Navigation Complete  
**Last Updated**: August 25 2025

## Navigation Philosophy

**"Simple flows for what exists, room for what's next."**

Navigation follows our core principle: ship what works today, add complexity when needed. Every flow documented here is **actually implemented**, not aspirational.

## Navigation State Machine

### Mode-Based Interaction
```
┌─────────────┐     [Space Key]     ┌──────────────┐
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

## Screen Hierarchy (Floating UI System)

```
RealityViewportApp
├── ContentView (SIMPLIFIED ARCHITECTURE) ✅
│   └── ZStack (Edge-to-Edge)
│       ├── ViewportStack (Full Screen)
│       │   ├── MetalSkyView (GPU Sky) ✅
│       │   ├── ViewportMetalGrid (GPU Grid) ✅
│       │   └── ViewportView (RealityKit) ✅
│       │
│       └── Floating UI Layer
│           ├── ViewportToolbar (Top, centered) ✅
│           ├── Inspector (Right, floating) ✅
│           └── StatusBar (Bottom, floating) ✅
```

### What Changed (August 2025)
```yaml
Before (Complex):
  - NavigationSplitView for iPad/Mac
  - NavigationStack for iPhone
  - Adaptive layouts with different paths
  - Inspector in sidebar (left)
  - Double toolbar rendering

After (Simple):
  - Single ZStack for all platforms ✅
  - Edge-to-edge viewport ✅
  - Floating glass panels ✅
  - Inspector on right (floating) ✅
  - Single toolbar (no duplication) ✅
  
Benefits:
  - Cleaner navigation (~100 lines removed)
  - Same experience on all platforms
  - More viewport space
  - Modern aesthetic
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
START → Edge-to-Edge Viewport → See Sky + Grid
   ↓
Floating Toolbar → Add Menu → Choose Entity Type
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
Toolbar: Folder Icon → ProjectBrowserView (Sheet)
   ├─→ New: Creates empty scene ✅
   ├─→ Open: Loads .rvproject ✅
   ├─→ Save: Serializes entities ✅
   └─→ Recent: Quick access ✅

Status Bar: Shows current project name ✅

Not Built Yet:
   ⏳ Templates (YAGNI)
   ⏳ Cloud sync (YAGNI)
   ⏳ Version control (YAGNI)
```

### 💾 Import Model Flow (Simplified)
```
Toolbar: Import Icon → Select USDZ/Reality
   ↓
ModelEntity created (wrapper)
   ↓
entity.load(from: url) // Async
   ↓
Appears at origin → Done

That's it. No complex import settings.
No material editors. It just works.
```

## Floating UI Navigation

### Inspector Toggle Flow
```
Toolbar: Sidebar Icon
   ↓
Inspector slides in from right (animated)
   ├─→ Outliner Tab: Scene hierarchy
   └─→ Properties Tab: Selected entity

Hide: Click X or sidebar icon again
   ↓
Inspector slides out (animated)
   ↓
More viewport space

Position: Always right side (floating)
Width: Fixed 320pt (on macOS) / 280pt (on iOS)
```

### Floating Panel Interactions
```yaml
Toolbar (Top):
  - Centered horizontally
  - 20pt padding from edges
  - Glass morphism (.ultraThinMaterial)
  - 12pt rounded corners
  - Subtle shadow for depth

Inspector (Right):
  - Slides in/out from right edge
  - 20pt from right when visible
  - 70pt from top (below toolbar)
  - Resizable properties section

Status Bar (Bottom):
  - Project info on left
  - Statistics on right
  - 20pt padding from edges
  - Minimal height (auto-sizing)
```

## Platform Navigation Patterns

### Unified Layout (All Platforms)
```yaml
All Platforms Now Use:
  Layout: ZStack with floating panels
  Inspector: Right-side floating panel
  Toolbar: Top floating glass bar
  Status: Bottom floating info bar
  
Platform Differences:
  macOS:
    - Mouse/trackpad gestures
    - Keyboard shortcuts
    - Hover states
    
  iOS:
    - Touch gestures
    - Haptic feedback
    - Timer-based updates
    
  tvOS:
    - Remote control
    - Focus engine
    - Simplified features

No more adaptive complexity!
```

### Input Mapping

#### macOS (What's Built)
| Input | Mode | Result |
|-------|------|--------|
| Click + Drag | Environment | Orbit camera ✅ |
| Right Click + Drag | Any | Pan camera ✅ |
| Scroll | Any | Zoom ✅ |
| Pinch (trackpad) | Any | Zoom ✅ |
| Click | Entity | Select ✅ |
| Space | Any | Toggle mode ✅ |

#### iOS (What's Built)
| Gesture | Mode | Result |
|---------|------|--------|
| One Finger Drag | Environment | Orbit ✅ |
| Two Finger Drag | Any | Pan ✅ |
| Pinch | Any | Zoom ✅ |
| Tap | Entity | Cycle selection ✅ |
| Long Press | Entity | Context menu ✅ |

### iOS-Specific Navigation Fix
```yaml
Problem Solved:
  - SwiftUI publishing errors during updates
  
Solution:
  - Timer-based updates (60fps)
  - Avoids view update conflicts
  - Smooth interaction maintained
  
Code:
  #if os(iOS)
  Timer.scheduledTimer(withTimeInterval: 1/60) {
    Task { @MainActor in
      updateEntities()
      updateCamera()
      updateGizmo()
    }
  }
  #endif
```

## State Management

### Simple State, Clear Ownership

```swift
// SceneManager owns entities (your wrappers)
@Published var entities: [Entity] = []

// SelectionManager owns selection (your wrappers)
@Published var selectedEntity: Entity?

// ViewportState owns rendering (RealityKit.Entity)
let rootEntity = RealityKit.Entity()

// ContentView owns UI state
@State private var showInspector = true  // Floating panel visibility
```

### Navigation State
```yaml
Current Implementation:
  viewport_mode: .environment | .entity ✅
  camera_state: distance, azimuth, elevation ✅
  selection: single entity ✅
  gizmo: visible when selected ✅
  inspector: show/hide (right side) ✅
  toolbar: always visible (top) ✅

Not Built (YAGNI):
  ⏳ Multi-selection UI
  ⏳ Selection groups
  ⏳ Navigation history
  ⏳ Saved views
  ⏳ Dockable panels
```

## Navigation Flows

### Creating an Entity
```
1. User: Clicks + in toolbar → Selects type
2. System: Creates Entity wrapper
3. System: Adds to SceneManager
4. Bridge: wrapper.realityEntity → ViewportState
5. View: Entity appears in viewport
6. Inspector: Updates outliner
```

### Selecting an Entity
```
1. User: Taps/clicks in viewport (Entity mode)
2. System: Cycles to next entity (iOS) or hit test (macOS)
3. Update: SelectionManager notified
4. Gizmo: Appears at entity position
5. Inspector: Shows properties
```

### Toggling Inspector
```
1. User: Clicks sidebar icon in toolbar
2. Animation: Inspector slides in/out from right
3. State: showInspector toggles
4. Viewport: Adjusts to fill space
5. Memory: State persists in session
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

iOS Publishing Error:
  Fixed: Timer-based updates
  No user-facing errors

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
  Inspector toggle: Smooth animation
  iOS updates: Consistent 60fps

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
  ❌ Dockable panels (floating works great)
  ❌ Multiple viewports (one is enough)
  ❌ Split views (edge-to-edge better)

Why: YAGNI - Add when games need them
```

### Future Navigation (If Needed)

```swift
// TODAY: Beautiful floating UI
// FUTURE: Add complexity only if requested

// Example: Docking (if users really want it)
extension Inspector {
    var dockingSide: DockSide = .floating  // Add later
}

// Example: Multiple views (if needed)
extension ViewportState {
    var additionalViews: [ViewportView] = []  // Add later
}
```

## Navigation Best Practices

### DO: Embrace Simplicity
```swift
// Good - Direct navigation with floating UI
ZStack {
    viewport
    toolbar
    inspector
}

// Over-engineered - Don't do this
NavigationSplitView {
    // Complex adaptive layouts
}
```

### DO: Keep Floating UI Consistent
```swift
// Good - Consistent styling
.background(.ultraThinMaterial)
.clipShape(RoundedRectangle(cornerRadius: 12))
.shadow(color: .black.opacity(0.2), radius: 10)
.padding(20)

// Bad - Mixed styles
.background(.regularMaterial)  // Different material
.cornerRadius(8)  // Different radius
```

### DON'T: Add Navigation Complexity
```swift
// Don't build navigation for:
- Features you haven't built
- Workflows users haven't requested  
- Multiple UI paradigms
- Platform-specific layouts (unless necessary)
```

## Platform-Specific Notes

### macOS
- Menu bar works ✅
- Keyboard shortcuts work ✅
- Floating UI beautiful ✅
- No docking needed (YAGNI)

### iOS/iPadOS
- Touch works ✅
- Gestures work ✅
- Haptics work ✅
- Timer-based updates ✅
- Same floating UI as Mac ✅

### tvOS
- Basic navigation ✅
- Limited testing ⚠️
- Simplified features
- Remote control works

## Visual Hierarchy

```yaml
Layer Order (back to front):
  1. Metal Sky (background)
  2. Metal Grid
  3. RealityKit Viewport
  4. Floating UI Elements
     - Toolbar (top)
     - Inspector (right)
     - Status (bottom)

Glass Effect Stack:
  - .ultraThinMaterial (consistent)
  - 12pt corner radius (everywhere)
  - Subtle shadows (depth)
  - 20pt padding (spacing)
```

## See Also
- **Implementation.md** - Floating UI details (v4.0)
- **Visual.md** - Glass morphism design
- **Architecture.md** - Overall philosophy
- **ViewportState.md** - Rendering connection

## Summary

Navigation v5.0 is **beautifully simple**:
- ✅ **Edge-to-edge viewport** - Maximum space for 3D
- ✅ **Floating glass UI** - Modern, consistent, beautiful
- ✅ **Single architecture** - ZStack for all platforms
- ✅ **Two modes** - Environment and Entity (enough)
- ✅ **Clear flows** - Create, select, transform, save
- ✅ **Room to grow** - Add navigation as features emerge

**Philosophy**: Simple navigation for a simple editor. Float above complexity.