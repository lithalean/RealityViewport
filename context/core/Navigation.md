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
   ├─→ New Project → Name Dialog → Location Picker → Create
   ├─→ Open Project → File Browser → Select .rvproject → Load
   ├─→ Recent Projects → Quick Access List → Select → Open
   └─→ Save As → Name Dialog → Location Picker → Save

Save Flow Details:
   First Save → Save As Dialog → Choose Name/Location
   Subsequent Save → Direct Save (Cmd+S)
   Save Copy As → New Name Dialog → Keep Original Open
```

### 💾 Import/Export Workflows

#### Import Multiple Files
```
Import Button → File Browser (Multi-Select)
   ↓
Select Multiple USDZ/Reality Files
   ↓
Import Progress (Per File)
   ├─→ Success: Add to Scene at Origin
   ├─→ Error: Show in Error List
   └─→ Complete: Select Last Imported
```

#### Export with Format Selection
```
Export Button → Format Picker Popover
   ├─→ USDZ: Options → Include Textures? → Save Dialog
   ├─→ Reality: Options → AR Optimized? → Save Dialog
   └─→ JSON: Options → Pretty Print? → Save Dialog
   
Post-Export:
   → Success Toast → Optional "Open in Finder"
```

### 🎮 Mode Switching Behavior

#### Environment → Entity Mode
```
Environment Mode (Camera Control)
   ↓ [Space or Mode Toggle]
Entity Mode Activated
   ├─→ Gizmos Become Visible
   ├─→ Camera Lock Indicator
   ├─→ Selection Highlighting Enabled
   └─→ Cursor Changes to Select
```

#### Entity → Environment Mode  
```
Entity Mode (Object Manipulation)
   ↓ [Space or Mode Toggle]
Environment Mode Activated
   ├─→ Gizmos Hidden
   ├─→ Camera Unlock Indicator
   ├─→ Selection Dimmed
   └─→ Cursor Changes to Orbit
```

### 🔧 Gizmo Interaction Patterns

#### Gizmo Selection Flow
```
Entity Selected → Gizmo Appears at Entity
   ↓
Hover Over Axis → Axis Highlights
   ↓
Click + Drag → Constrained Movement
   ├─→ X Axis: Red Line Guide
   ├─→ Y Axis: Green Line Guide
   ├─→ Z Axis: Blue Line Guide
   └─→ Plane: Grid Guide
   
Release → Update Transform → Save Undo State
```

#### Multi-Axis Interaction
```
Center Sphere Hover → All Axes Highlight
   ↓
Drag → Free Movement on Screen Plane
   ↓
Shift + Drag → Snapping Enabled (1 unit increments)
```# RealityViewport Navigation Context

**Purpose**: Screen flow, user journeys, and state management  
**Version**: 2.0  
**Navigation Pattern**: Mode-Based Tool System with File Operations  
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
1. User clicks Import (+) or File → Import
2. System file browser opens with filters
3. User selects .usdz/.reality file(s)
4. Security scope accessed
5. Loading progress indicator shows
6. Model validation occurs
7. On Success:
   - Model appears at origin
   - Model auto-selected
   - Properties panel updates
   - Success haptic (iOS)
8. On Error:
   - Error alert with details
   - Option to try different file
```

### Save Project Flow
```
1. User clicks Save (Cmd+S)
2. If new project:
   - Name dialog appears
   - User enters project name
   - Location picker shows
   - Default: ~/Documents/RealityViewport Projects
3. Create .rvproject directory structure
4. Save progress indicator
5. Copy referenced assets to project
6. Update window title with project name
7. Enable incremental save
```

### New Project with Naming
```
File → New Project
   ↓
Modal Sheet Appears
   ├─→ Project Name Field (Required)
   ├─→ Location Picker (Browse button)
   ├─→ Template Selection (Future)
   └─→ Create/Cancel Buttons
   
Validation:
   - Non-empty name
   - Valid filesystem characters
   - Unique in target directory
   
On Create:
   - Close current project (with save prompt)
   - Create project structure
   - Open new empty scene
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

## Platform-Specific Navigation

### macOS Navigation Features
```yaml
menu_bar_integration:
  - File menu with all project operations
  - Edit menu with transform tools
  - View menu with viewport options
  - Window menu with panel toggles

keyboard_navigation:
  - Tab through UI elements
  - Arrow keys for list navigation
  - Space for button activation
  - Escape to cancel modals

context_menus:
  - Right-click on entities
  - Right-click in viewport
  - Right-click in outliner

window_management:
  - Resizable inspector panels
  - Detachable panels (future)
  - Full screen mode support
```

### iOS Navigation Adaptations
```yaml
gesture_shortcuts:
  - Three-finger tap: Undo
  - Three-finger swipe: Redo
  - Two-finger double tap: Reset view
  - Long press: Context menu

touch_optimizations:
  - 44pt minimum touch targets
  - Expanded hit areas for gizmos
  - Touch-and-hold for precision
  - Haptic feedback on actions

modal_presentations:
  - Sheet style for file operations
  - Popover for quick options
  - Full screen for immersive edit
```

### tvOS Remote Navigation
```yaml
focus_engine:
  - Automatic focus guides
  - Focus rings on UI elements
  - Directional navigation

remote_mappings:
  - Swipe: Navigate/Orbit
  - Click: Select/Activate
  - Play/Pause: Mode toggle
  - Menu: Back/Context

simplified_ui:
  - Larger UI elements
  - Reduced information density
  - Clear focus indicators
```