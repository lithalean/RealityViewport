# RealityViewport Navigation Context

**Purpose**: Screen flow, user journeys, and state management  
**Version**: 1.0  
**Navigation Pattern**: Mode-Based Tool System  
**Last Updated**: July 2025

## Navigation State Machine
```
┌─────────────┐     Switch Mode    ┌──────────────┐
│   Camera    │ ◄─────────────────► │   Object     │
│    Mode     │                     │    Mode      │
└─────────────┘                     └──────────────┘
      │                                    │
      │ Gestures                          │ Select
      ▼                                    ▼
┌─────────────┐                     ┌──────────────┐
│   Orbit/    │                     │  Transform   │
│   Pan/Zoom  │                     │   Gizmos     │
└─────────────┘                     └──────────────┘
```

## Screen Hierarchy
```
RealityViewportApp
├── ContentView (Main Container)
│   ├── Platform Router
│   │   ├── MacView (macOS)
│   │   ├── iPhoneView (iOS)
│   │   └── ContentView (tvOS)
│   │
│   ├── ViewportView (3D Scene)
│   │   ├── RealityView
│   │   ├── ViewportToolbar
│   │   ├── ViewportAxisHelper
│   │   └── ViewportGrid
│   │
│   ├── InspectorView (Right Panel)
│   │   ├── OutlinerView
│   │   └── PropertiesView
│   │
│   └── Sheets/Modals
│       ├── ProjectBrowserView
│       ├── Import File Dialog
│       └── Export Options
```

## User Journey Flows

### 🎬 First Launch Journey
```
START → Empty Scene → Viewport Displayed → Grid Visible
   ↓
Add Menu → Choose Entity (Camera/Light) → Entity Created
   ↓
See Billboard Icon → Select Entity → View Properties
   ↓
Import Model → File Browser → USDZ Selected → Model Loaded
```

### 📁 Project Management Flow
```
Projects Button → ProjectBrowserView Sheet
   ├─→ New Project → Enter Name → Create
   ├─→ Open Project → Select .rvproject → Load
   └─→ Recent Projects → Quick Access → Open
```

### 🎮 Interaction Mode Flow
```
Camera Mode (Default)
   ├─→ Drag: Orbit camera
   ├─→ Right Drag: Pan camera  
   └─→ Scroll/Pinch: Zoom

[Toggle Mode Button]
   ↓
Object Mode
   ├─→ Click: Select object
   ├─→ Gizmo: Transform (future)
   └─→ Multi-select: Cmd/Shift (future)
```

### 🔧 Entity Creation Flow
```
Add Menu (+) → Entity Type Menu
   ├─→ Camera → New camera at origin
   ├─→ Light → New light at origin
   ├─→ Primitives (future)
   │     ├─→ Cube
   │     ├─→ Sphere
   │     └─→ Plane
   └─→ Import → File browser
```

## Navigation Quick Reference

| From | To | Trigger | Duration | Type |
|------|-----|---------|----------|------|
| Any View | ProjectBrowser | Projects button | 300ms | Sheet |
| Any View | Import Dialog | Import menu/button | 200ms | System |
| Camera Mode | Object Mode | Mode toggle | Instant | State |
| No Selection | Selected | Click entity | 200ms | Highlight |
| Outliner | Properties | Select item | Instant | Update |

## Gesture Navigation

### macOS
| Gesture | Mode | Action |
|---------|------|--------|
| Click + Drag | Camera | Orbit camera |
| Right Click + Drag | Camera | Pan camera |
| Scroll Wheel | Camera | Zoom |
| Click | Object | Select entity |
| Cmd + Click | Object | Multi-select (future) |
| Double Click | Any | Focus on object |

### iOS
| Gesture | Mode | Action |
|---------|------|--------|
| One Finger Drag | Camera | Orbit camera |
| Two Finger Drag | Camera | Pan camera |
| Pinch | Camera | Zoom |
| Tap | Object | Select entity |
| Long Press | Object | Context menu (future) |
| Double Tap | Any | Focus on object |

## State Management

### Global States
```swift
// App Level
@StateObject var sceneManager: SceneManager
@StateObject var projectManager: ProjectManager

// View Level  
@State var showProjectBrowser: Bool
@State var showImporter: Bool
@State var currentMode: InteractionMode
```

### Navigation States
```yaml
viewport_state:
  mode: "camera" | "object"
  camera_locked: false
  grid_visible: true
  gizmos_visible: true
  
selection_state:
  selected_nodes: []
  multi_select_enabled: false
  selection_locked: false
  
panel_state:
  inspector_visible: true
  inspector_tab: "outliner" | "properties"
  console_visible: false
```

## Modal Flows

### Import Model Flow
```
1. User clicks Import
2. System file browser opens
3. User selects .usdz/.reality file
4. Loading indicator shows
5. Model appears at origin
6. Model auto-selected
7. Properties panel updates
```

### Save Project Flow (Not Yet Connected)
```
1. User clicks Save
2. If new project:
   - Name dialog appears
   - User enters name
   - Choose location
3. Save progress indicator
4. Success feedback
5. Window title updates
```

## Error States

### Import Failures
```
Failed Import → Error Alert
   ├─→ Unsupported Format
   ├─→ Corrupted File
   └─→ Memory Issue
   
Recovery: Return to previous state
```

### Project Load Failures
```
Failed Load → Error Dialog
   ├─→ Missing Files
   ├─→ Version Mismatch
   └─→ Corrupted Project
   
Recovery: Offer to create new project
```

## Keyboard Navigation

### Global Shortcuts (macOS)
| Shortcut | Action |
|----------|---------|
| Cmd+N | New Project |
| Cmd+O | Open Project |
| Cmd+S | Save Project |
| Cmd+Z | Undo (future) |
| Delete | Delete Selected |
| Space | Toggle Mode |

### Navigation Shortcuts
| Key | Action |
|-----|---------|
| F | Focus on Selected |
| G | Toggle Grid |
| H | Toggle Gizmos |
| Tab | Next UI Element |

## Future Navigation Enhancements

### Planned Flows
- [ ] Drag & drop from Finder
- [ ] Quick switcher (Cmd+K)
- [ ] Breadcrumb navigation
- [ ] History navigation (back/forward)
- [ ] Workspace layouts
- [ ] Full screen mode