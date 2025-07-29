# RealityViewport Visual Design Context

**Purpose**: Complete visual design system and UI specifications  
**Version**: 1.0  
**Design Language**: Apple HIG + Custom 3D Elements  
**Last Updated**: July 2025

> ⚠️ **Note**: Design specifications inferred from implementation. Update with actual design values.

## Visual Hierarchy
```
Z-LAYERS (back→front):
├─[0] RealityKit Scene (3D viewport)
├─[1] Grid Helper (reference grid)
├─[2] Scene Entities (models, cameras, lights)
├─[3] Billboard Icons (always-facing)
├─[4] Transform Gizmos (selection tools)
├─[5] Viewport Overlay (axis helper)
└─[6] UI Layer (toolbar, inspector)
```

## Quick Reference Tables

### Color Palette
| Element | Light Mode | Dark Mode | Usage |
|---------|------------|-----------|-------|
| **Background** | systemBackground | systemBackground | Main viewport |
| **Grid** | gray.opacity(0.3) | gray.opacity(0.3) | Reference grid |
| **Selection** | accentColor | accentColor | Selected objects |
| **Gizmo X** | Color.red | Color.red | X-axis transform |
| **Gizmo Y** | Color.green | Color.green | Y-axis transform |
| **Gizmo Z** | Color.blue | Color.blue | Z-axis transform |
| **Icons** | label | label | Billboard icons |

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
height: 44pt (iOS) / 38pt (macOS)
background: .regularMaterial
items:
  - segmented_control: Mode selection
  - divider: 1pt separator
  - buttons: Tool actions
  - spacer: Flexible space
  - menu: Add entities
```

### Transform Gizmo
```yaml
style: "Standard 3D manipulation"
colors:
  x_axis: red
  y_axis: green  
  z_axis: blue
  center: white
  selected: yellow
scale: "Constant screen size"
opacity: 0.8
hover_opacity: 1.0
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

## Future Visual Considerations

### Planned Enhancements
- [ ] Custom viewport themes
- [ ] Adjustable grid colors
- [ ] Icon size preferences
- [ ] Gizmo style options
- [ ] Dark/light mode override