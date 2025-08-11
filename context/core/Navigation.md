# RealityViewport Navigation Context

**Purpose**: Screen flow, user journeys, and state management with Entity/ECS and Adaptive UI  
**Version**: 3.0  
**Navigation Pattern**: Mode-Based Tool System with Entity Operations  
**Last Updated**: August 2025

## Navigation State Machine (Entity-Based)
```
┌─────────────┐     Switch Mode    ┌──────────────┐
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

## Screen Hierarchy (Adaptive UI v3.0)
```
RealityViewportApp
├── ContentView (SINGLE ADAPTIVE VIEW)
│   ├── NavigationSplitView (Regular: iPad/Mac)
│   │   ├── Sidebar: InspectorView
│   │   └── Detail: ViewportStack
│   │
│   ├── NavigationStack (Compact: iPhone)
│   │   ├── Main: ViewportStack
│   │   └── Inspector: Sheet/Overlay
│   │
│   ├── ViewportStack (Composite Rendering)
│   │   ├── MetalSkyView (Layer 0: GPU Sky)
│   │   ├── ViewportMetalGrid (Layer 1: GPU Grid)
│   │   └── ViewportView (Layer 2: RealityKit)
│   │       ├── RealityView (Entities)
│   │       ├── ViewportToolbar
│   │       └── CameraController
│   │
│   ├── InspectorView (Adaptive Panel)
│   │   ├── OutlinerView (Entity Hierarchy)
│   │   └── PropertiesView (Entity Properties)
│   │
│   └── Sheets/Modals
│       ├── ProjectBrowserView
│       ├── Import File Dialog
│       └── Export Options
```

## User Journey Flows (Entity System)

### 🎬 First Launch Journey
```
START → Empty Scene → Metal Sky + Grid Visible
   ↓
Add Menu → Choose Entity Type (Camera/Light/Model)
   ↓
Entity Created → SceneManager.addEntity()
   ↓
See Billboard Icon → Select Entity → SelectionManager.select()
   ↓
View Properties → InspectorView Updates → Modify Entity
```

### 📁 Project Management Flow (Entity-Based)
```
Projects Button → ProjectBrowserView Sheet
   ├─→ New Project → Name Dialog → Location Picker → Create
   ├─→ Open Project → File Browser → Select .rvproject → Load Entities
   ├─→ Recent Projects → Quick Access List → Select → Deserialize Entities
   └─→ Save As → Name Dialog → Serialize Entities → Save

Entity Serialization:
   Entities → ProjectEntityData → JSON → .rvproject file
   Load: .rvproject → ProjectEntityData → Recreate Entities
```

### 💾 Import/Export Workflows (Entity System)

#### Import Model as Entity
```
Import Button → File Browser → Select USDZ/Reality
   ↓
Create ModelEntity → entity.isLoading = true
   ↓
Async Load: await entity.load(from: url)
   ↓
Add to Scene: sceneManager.addEntity(entity)
   ↓
Auto-Select: selectionManager.select(entity)
```

#### Export Entities
```
Export Button → Format Picker
   ├─→ JSON: Serialize all entities → ProjectSceneData
   ├─→ USDZ: Convert entities to RealityKit → Export (pending)
   └─→ Reality: Similar to USDZ (pending)
```

### 🎮 Mode Switching Behavior (ViewportState)

#### Environment → Entity Mode
```
Environment Mode (Camera Control)
   ↓ [Space or Segmented Control]
viewportState.interactionMode = .entity
   ├─→ Gizmos Enabled for Selected Entity
   ├─→ Camera Lock Visual Indicator
   ├─→ Entity Selection Active
   └─→ Cursor Changes to Select
```

#### Entity → Environment Mode  
```
Entity Mode (Entity Manipulation)
   ↓ [Space or Segmented Control]
viewportState.interactionMode = .environment
   ├─→ Gizmos Disabled
   ├─→ Camera Unlock Indicator
   ├─→ Camera Controls Active
   └─→ Cursor Changes to Orbit
```

### 🔧 Entity Transform Patterns

#### Entity Selection Flow
```
Click in Viewport → Hit Test → Find RealityKit.Entity
   ↓
Map to Custom Entity via SceneManager
   ↓
selectionManager.select(entity)
   ├─→ Gizmo Created at entity.position
   ├─→ Properties Panel Updates
   ├─→ Outliner Highlights
   └─→ Selection Component Added
```

#### Gizmo Interaction with Entity
```
Entity Selected → TransformGizmo at entity.position
   ↓
Hover Over Axis → Axis Highlights
   ↓
Drag → Update entity.position/rotation/scale
   ├─→ X Axis: entity.position.x changes
   ├─→ Y Axis: entity.position.y changes
   ├─→ Z Axis: entity.position.z changes
   └─→ Center: Free movement
   
Release → entity.realityEntity syncs → Scene Updates
```

## Adaptive Layout Behaviors

### NavigationSplitView (Regular Width)
```
iPad/Mac Layout:
┌──────────────────────────────────────┐
│ ┌─────────┬────────────────────────┐ │
│ │Inspector│      Viewport           │ │
│ │         │   ┌──────────────┐     │ │
│ │Outliner │   │ Metal + RK   │     │ │
│ │─────────│   │   Layers     │     │ │
│ │Properties│  └──────────────┘     │ │
│ └─────────┴────────────────────────┘ │
└──────────────────────────────────────┘
```

### NavigationStack (Compact Width)
```
iPhone Layout:
┌─────────────┐
│  Viewport   │
│ ┌─────────┐ │
│ │ Metal + │ │
│ │   RK    │ │
│ └─────────┘ │
│             │
│ [Toolbar]   │  ← Floating bottom
│ [Inspector] │  ← Sheet overlay
└─────────────┘
```

## Navigation Quick Reference (Entity System)

| From | To | Trigger | State Change | Type |
|------|-----|---------|--------------|------|
| Any View | ProjectBrowser | Projects button | Load entities | Sheet |
| Any View | Import Dialog | Import menu | Create ModelEntity | System |
| Environment | Entity Mode | Mode toggle | viewportState.interactionMode | Instant |
| No Selection | Entity Selected | Click entity | selectionManager.select() | Highlight |
| Outliner | Properties | Select entity | @Published updates | Instant |

## Gesture Navigation (Entity Operations)

### macOS
| Gesture | Mode | Action | Entity Impact |
|---------|------|--------|---------------|
| Click + Drag | Environment | Orbit camera | None |
| Right Click + Drag | Environment | Pan camera | None |
| Scroll Wheel | Environment | Zoom | None |
| Click | Entity | Select entity | SelectionManager updates |
| Cmd + Click | Entity | Multi-select | Add to selection array |
| Double Click | Any | Focus on entity | Camera targets entity |

### iOS
| Gesture | Mode | Action | Entity Impact |
|---------|------|--------|---------------|
| One Finger Drag | Environment | Orbit camera | None |
| Two Finger Drag | Environment | Pan camera | None |
| Pinch | Environment | Zoom | None |
| Tap | Entity | Select entity | SelectionManager updates |
| Long Press | Entity | Context menu | Entity options |
| Double Tap | Any | Focus on entity | Camera targets entity |

## State Management (Entity System)

### Manager States
```swift
// App Level - Environment Objects
@StateObject var sceneManager: SceneManager      // Entity management
@StateObject var projectManager: ProjectManager  // File operations
@StateObject var dayNightManager: DayNightManager // Atmosphere

// SceneManager State
@Published var entities: [any SceneEntity] = []
@Published var selectedEntity: (any SceneEntity)?

// ViewportState (RealityKit.Entity management)
let rootEntity = RealityKit.Entity()
@Published var interactionMode: ViewportInteractionMode
@Published var needsUpdate: Bool
```

### Navigation States
```yaml
viewport_state:
  mode: .environment | .entity
  camera_distance: Float
  camera_azimuth: Float
  camera_elevation: Float
  grid_visible: true (Metal rendered)
  
selection_state:
  selected_entities: [any SceneEntity]
  primary_selection: SceneEntity?
  gizmo_mode: .translate | .rotate | .scale
  
inspector_state:
  visibility: .automatic | .visible | .hidden
  properties_height: CGFloat
  active_tab: .outliner | .properties
```

## Modal Flows (Entity Context)

### Import Model Entity Flow
```
1. User clicks Import (+) or File → Import
2. System file browser with .usdz/.reality filters
3. User selects file(s)
4. For each file:
   - Create ModelEntity(modelNamed: filename)
   - Set entity.isLoading = true
   - sceneManager.addEntity(entity)
   - await entity.load(from: url) // Async
   - entity.isLoading = false
5. On Success:
   - Entity appears at origin
   - Auto-select last imported
   - Properties panel updates
6. On Error:
   - Show error alert
   - Remove failed entity
```

### Save Project with Entities
```
1. User triggers Save (Cmd+S)
2. Serialize entities:
   - Convert each entity to ProjectEntityData
   - Include transform, type-specific data
   - Create ProjectSceneData
3. If new project:
   - Name dialog appears
   - Location picker (default: ~/Documents)
4. Write JSON to .rvproject
5. Update window title
6. Add to recent projects
```

## Platform-Specific Navigation

### Adaptive ContentView Behavior
```yaml
size_class_detection:
  compact: < 600pt width
  regular: >= 600pt width
  
layout_switching:
  regular:
    type: NavigationSplitView
    inspector: sidebar (320pt)
    toolbar: top_leading
    
  compact:
    type: NavigationStack
    inspector: sheet overlay
    toolbar: bottom_floating
```

### macOS Features
```yaml
menu_bar:
  File: New/Open/Save with entities
  Edit: Entity operations
  View: Viewport options, day/night
  Add: Create entities submenu

keyboard:
  Cmd+N: New project (clear entities)
  Cmd+S: Save entities
  Delete: Remove selected entity
  Space: Toggle interaction mode
```

### iOS Adaptations
```yaml
gestures:
  entity_selection: tap with hit test
  gizmo_interaction: direct manipulation
  context_menu: long press on entity
  
haptics:
  entity_created: .light
  entity_selected: .selection
  mode_changed: .medium
```

## Error Recovery

### Entity Load Failures
```
Failed ModelEntity Load → Error Alert
   ├─→ Invalid Format: Remove entity
   ├─→ Missing File: Offer re-link
   └─→ Memory Issue: Suggest smaller model
   
Recovery: Fallback to placeholder cube
```

### Project Entity Deserialization
```
Failed Entity Recreation → Warning Log
   ├─→ Unknown type: Skip entity
   ├─→ Corrupt data: Use defaults
   └─→ Missing assets: Placeholder
   
Recovery: Load partial scene
```

## Keyboard Shortcuts (Entity Operations)

### Global (All Platforms)
| Shortcut | Action | Entity Impact |
|----------|--------|---------------|
| Space | Toggle Mode | Change interaction |
| Delete | Delete Entity | Remove from scene |
| F | Focus Entity | Frame selected |
| G | Toggle Grid | Metal grid visibility |
| D | Duplicate | Clone selected entity |

### Transform (Entity Mode)
| Key | Action | Entity Change |
|-----|--------|---------------|
| W | Move Tool | Gizmo mode |
| E | Rotate Tool | Gizmo mode |
| R | Scale Tool | Gizmo mode |
| Q | Select Tool | No gizmo |

## Day/Night Cycle Navigation

### Time Control
```
Automatic Cycle (Default)
   ├─→ 2 minute full cycle
   ├─→ Smooth phase transitions
   └─→ Updates Metal sky shader

Manual Control (Debug)
   ├─→ Slider in debug panel
   ├─→ Jump to phase buttons
   └─→ Pause/resume cycle
```

## Summary

Navigation v3.0 fully embraces:
- ✅ **Entity/ECS system** - All operations on entities, not nodes
- ✅ **Adaptive UI** - Single ContentView for all platforms
- ✅ **Metal rendering** - GPU-accelerated viewport layers
- ✅ **Async operations** - Non-blocking model loading
- ✅ **Type-safe managers** - Clear separation of concerns

The navigation system provides intuitive flows for entity manipulation while maintaining platform-appropriate interactions.