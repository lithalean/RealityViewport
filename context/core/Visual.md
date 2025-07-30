# RealityViewport Visual Design Context

**Purpose**: Complete visual design system and UI specifications  
**Version**: 2.0  
**Design Language**: Apple HIG + Custom 3D Elements  
**Last Updated**: July 2025

## Visual Hierarchy
```
Z-LAYERS (back→front):
├─[0] RealityKit Scene (3D viewport)
├─[1] Grid Helper (reference grid)
├─[2] Scene Entities (models, cameras, lights)
├─[3] Billboard Icons (always-facing)
├─[4] Transform Gizmos (selection tools)
├─[5] Viewport Overlay (axis helper)
└─[6] UI Layer (toolbar, inspector, dialogs)
```

## Quick Reference Tables

### Color Palette
| Element | Light Mode | Dark Mode | Hex/Usage |
|---------|------------|-----------|-----------|
| **Background** | systemBackground | systemBackground | Main viewport |
| **Grid** | gray.opacity(0.3) | gray.opacity(0.3) | Reference grid |
| **Selection** | accentColor | accentColor | Selected objects |
| **Gizmo X** | .red | .red | X-axis transform |
| **Gizmo Y** | .green | .green | Y-axis transform |
| **Gizmo Z** | .blue | .blue | Z-axis transform |
| **Gizmo XY** | .yellow | .yellow | XY plane |
| **Gizmo XZ** | .cyan | .cyan | XZ plane |
| **Gizmo YZ** | .purple | .purple | YZ plane |
| **Gizmo Center** | .white | .gray | Multi-axis |
| **Button Active** | accentColor.opacity(0.2) | accentColor.opacity(0.2) | Active state |
| **Icons** | .label | .label | Billboard icons |
| **Error** | .red | .red | Error states |
| **Success** | .green | .green | Success feedback |

### Spacing System
| Token | Value | Usage |
|-------|-------|-------|
| **small** | 8pt | Internal padding |
| **medium** | 16pt | Component spacing |
| **large** | 24pt | Section gaps |
| **toolbar** | 12pt | Toolbar items |

### Typography Scale
| Style | Size | Weight | Usage |
|-------|------|--------|-------|
| **Title** | .title2 | .medium | Panel headers |
| **Body** | .body | .regular | General text |
| **Caption** | .caption | .regular | Help text |
| **Tool** | .footnote | .medium | Toolbar labels |

## Component Specifications

### Viewport Toolbar
```yaml
height: 
  macOS: 38pt
  iOS: 44pt
  tvOS: 60pt
background: .regularMaterial
layout: horizontal
padding: 12pt
items:
  - mode_toggle:
      type: segmented_control
      options: ["Environment", "Entity"]
      style: bordered
  - divider:
      width: 1pt
      color: .separator
  - tool_buttons:
      spacing: 8pt
      style: bordered_prominent (when active)
  - spacer:
      type: flexible
  - add_menu:
      style: menu_button
      icon: "plus.circle.fill"
responsive_behavior:
  compact: hide_labels
  regular: show_labels
```

### Transform Gizmo Specifications
```yaml
style: "3D Axis-Aligned"
components:
  axes:
    length: 2.0 units
    thickness: 0.05 units
    tip_size: 0.15 units
  planes:
    size: 0.5 units
    opacity: 0.3
  center:
    size: 0.2 units
    style: sphere
colors:
  x_axis: red
  y_axis: green  
  z_axis: blue
  xy_plane: yellow
  xz_plane: cyan
  yz_plane: purple
  center: white/gray
  hover: 20% brighter
  active: 40% brighter
interaction_states:
  idle:
    opacity: 0.8
    scale: 1.0
  hover:
    opacity: 1.0
    scale: 1.1
    cursor: platform_specific
  dragging:
    opacity: 1.0
    scale: 1.0
    show_guides: true
screen_space_scaling: true
minimum_size: 60pt
maximum_size: 200pt
```

### Billboard Icons
```yaml
size: 24x24pt
background: "Semi-transparent circle"
style: "SF Symbols"
icons:
  camera: "camera.fill"
  light: "light.max"
  model: "cube.fill"
  empty: "circle.dotted"
```

### Inspector Panel
```yaml
width: 
  min: 250pt
  ideal: 300pt
  max: 400pt
sections:
  - outliner: "Scene hierarchy"
  - properties: "Selected object properties"
style: "Grouped list with disclosure"
```

## Platform-Specific Adaptations

### macOS
- Smaller toolbar height (38pt)
- Hover states on all interactive elements
- Keyboard shortcuts visible
- Right-click context menus

### iOS
- Standard toolbar height (44pt)
- Touch-friendly tap targets (44x44pt min)
- Gesture hints on first launch
- Haptic feedback on selection

### tvOS
- Focus-based navigation
- Larger UI elements
- Simplified toolbar
- Remote-friendly controls

## Visual Effects

### Materials
```yaml
viewport_background:
  type: "Solid color"
  value: "System background"
  
toolbar_material:
  type: "UIBlurEffect"
  style: ".regularMaterial"
  
inspector_background:
  type: "Grouped style"
  style: ".grouped"
```

### Animations
```yaml
selection_highlight:
  duration: 0.2s
  curve: "easeInOut"
  
gizmo_hover:
  scale: 1.2x
  duration: 0.15s
  
panel_toggle:
  type: "slide"
  duration: 0.3s
```

## Icon System

### Entity Icons (SF Symbols)
| Entity Type | Icon | Size | Color |
|-------------|------|------|-------|
| Camera | camera.fill | 20pt | .label |
| Light | light.max | 20pt | .yellow |
| Model | cube.fill | 20pt | .label |
| Group | folder.fill | 20pt | .label |

### Tool Icons
| Tool | Icon | Size | State |
|------|------|------|-------|
| Select | cursorarrow | 16pt | Toggle |
| Move | move.3d | 16pt | Toggle |
| Rotate | rotate.3d | 16pt | Toggle |
| Scale | arrow.up.left.and.arrow.down.right | 16pt | Toggle |

## Design Tokens

### Shadows
```yaml
gizmo_shadow:
  color: black.opacity(0.3)
  radius: 4
  offset: (0, 2)
  
panel_shadow:
  color: black.opacity(0.1)
  radius: 8
  offset: (0, 4)
```

### Corner Radius
```yaml
panels: 8pt
buttons: 6pt
toolbar: 0pt
icons: circle
```

## Accessibility

### High Contrast Mode
- Increased grid opacity
- Thicker selection outlines
- Higher contrast gizmo colors

### Reduced Motion
- Instant transitions
- No hover animations
- Static billboard rotation

## File Dialog Visual Flows

### New Project Dialog
```yaml
type: modal_sheet
width: 400pt
content:
  - title: "New Project"
  - text_field:
      label: "Project Name"
      placeholder: "My Project"
  - location_picker:
      label: "Save Location"
      default: "~/Documents/RealityViewport Projects"
  - buttons:
      cancel: secondary_style
      create: primary_style
validation:
  - non_empty_name
  - valid_characters
  - unique_name_in_directory
```

### Import Dialog Flow
```yaml
stages:
  file_selection:
    type: system_file_browser
    allowed_types: [.usdz, .reality]
    multiple_selection: true
  import_progress:
    type: progress_sheet
    shows: file_name, progress_bar, cancel_button
  completion:
    type: auto_dismiss
    duration: 1.5s
    shows: success_checkmark
```

### Export Format Selection
```yaml
type: popover
trigger: export_button
content:
  - section_header: "Export Format"
  - radio_group:
      options:
        - USDZ (3D Scene)
        - Reality (AR Quick Look)
        - JSON (Scene Data)
  - section_header: "Options"
  - checkboxes:
      - Include Textures
      - Optimize Geometry
  - export_button: primary_style
```