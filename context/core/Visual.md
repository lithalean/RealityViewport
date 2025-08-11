# RealityViewport Visual Design System

**Purpose**: Complete visual design system, UI specifications, and adaptive layouts  
**Version**: 3.0  
**Design Language**: Apple HIG + Metal Rendering + Adaptive UI  
**Last Updated**: August 2025

## Visual Architecture
```
RENDERING LAYERS (back→front):
├─[0] Metal Sky (Dynamic gradient background)
├─[1] Metal Grid (GPU-accelerated reference grid)
├─[2] RealityKit Scene (3D models & entities)
├─[3] Billboard Icons (Camera-facing UI elements)
├─[4] Transform Gizmos (Manipulation tools)
├─[5] Viewport Overlays (Axis helper, stats)
└─[6] SwiftUI Layer (Adaptive UI, inspector, toolbar)
```

## Adaptive Layout System (WWDC25 Standard)

### Layout Strategy
```swift
// Single ContentView adapts to all contexts
NavigationSplitView (Regular: iPad/Mac)
├─ Sidebar: Inspector (collapsible)
└─ Detail: Viewport + Toolbar

NavigationStack (Compact: iPhone)
├─ Main: Viewport
├─ Toolbar: Bottom floating
└─ Inspector: Sheet/Overlay
```

### Size Class Breakpoints
| Class | Width | Platform | Layout |
|-------|-------|----------|--------|
| **Compact** | < 600pt | iPhone, iPad Split | Stack layout, bottom toolbar |
| **Regular** | ≥ 600pt | iPad, Mac | Split view, inline inspector |
| **tvOS** | Full | Apple TV | Focus-based, simplified |

### Adaptive Components

#### Toolbar Adaptation
```yaml
regular_layout:
  position: top_leading_overlay
  style: bordered_prominent
  spacing: 12pt
  shows_labels: true

compact_layout:
  position: bottom_floating
  style: material_background
  spacing: 8pt
  shows_labels: false
  padding: safe_area + 8pt
```

#### Inspector Adaptation
```yaml
regular_layout:
  style: NavigationSplitView.sidebar
  width: 320pt
  resizable: true (280-400pt)
  position: leading_side

compact_layout:
  style: .inspector() modifier
  presentation: slide_over
  width: 80% of screen
  dismissable: true
```

## Metal Rendering Visual Specs

### Sky Gradient System (Day/Night Cycle)
```yaml
dawn:
  top: rgb(102, 128, 179)    # Soft blue-gray
  bottom: rgb(230, 153, 102)  # Warm orange
  duration: 0.25 cycle

day:
  top: rgb(77, 153, 230)      # Sky blue
  bottom: rgb(179, 217, 255)  # Light blue
  duration: 0.25 cycle

dusk:
  top: rgb(77, 51, 128)       # Purple
  bottom: rgb(204, 102, 77)   # Orange-red
  duration: 0.25 cycle

night:
  top: rgb(13, 13, 38)        # Dark blue
  bottom: rgb(26, 26, 64)     # Midnight blue
  duration: 0.25 cycle

transition: smooth_interpolation
cycle_time: 120_seconds_default
```

### Grid Rendering
```yaml
style: metal_shader_based
lines:
  major:
    color: rgba(0.3, 0.3, 0.3, 0.5)
    width: 1.0pt
    spacing: 1.0_unit
  minor:
    color: rgba(0.2, 0.2, 0.2, 0.3)
    width: 0.5pt
    spacing: 0.1_unit
axes:
  x: red (1.0, 0.0, 0.0, 1.0)
  y: green (0.0, 1.0, 0.0, 1.0)
  z: blue (0.0, 0.0, 1.0, 1.0)
  width: 2.0pt
fade:
  start_distance: 5.0_units
  end_distance: 20.0_units
  function: smooth_step
```

## Updated Color System

### Core Palette
| Element | Light Mode | Dark Mode | Metal/Hex | Usage |
|---------|------------|-----------|-----------|--------|
| **Sky** | Dynamic | Dynamic | See cycle above | Background gradient |
| **Grid** | 0.3 alpha gray | 0.3 alpha gray | #4C4C4C80 | Reference grid |
| **Grid Fade** | Distance-based | Distance-based | Shader calc | Depth cue |
| **Selection** | accentColor | accentColor | System | Selected objects |
| **Gizmo X** | .red | .red | #FF0000 | X-axis |
| **Gizmo Y** | .green | .green | #00FF00 | Y-axis |
| **Gizmo Z** | .blue | .blue | #0000FF | Z-axis |
| **Material** | .regularMaterial | .regularMaterial | System | UI panels |

### Entity Type Colors
| Type | Icon Color | Selection Tint |
|------|------------|----------------|
| Camera | .blue | .blue.opacity(0.2) |
| Light | .yellow | .yellow.opacity(0.2) |
| Model | .purple | .purple.opacity(0.2) |
| Empty | .gray | .gray.opacity(0.2) |

## Component Specifications

### Unified Viewport Toolbar
```yaml
adaptive_behavior:
  compact:
    container: floating_capsule
    position: bottom - 8pt
    background: .regularMaterial
    corner_radius: 12pt
  regular:
    container: inline_overlay
    position: top_leading + 16pt
    background: .regularMaterial
    corner_radius: 8pt

controls:
  mode_selector:
    type: segmented_control
    options: ["Environment", "Entity"]
    width: adaptive
  
  gizmo_tools:
    type: button_group
    items: [move, rotate, scale]
    style: bordered_when_active
    
  viewport_actions:
    type: menu
    icon: plus.circle.fill
    items: [add_camera, add_light, add_primitive]
```

### Transform Gizmo (Metal + RealityKit Hybrid)
```yaml
rendering: RealityKit entities
scaling: screen_space_constant
size:
  base: 0.1 * camera_distance
  min: 60pt_screen_space
  max: 200pt_screen_space
  
interaction:
  hover:
    scale: 1.1x
    brightness: 1.2x
    cursor: platform_specific
  
  drag:
    constraint_display: true
    axis_highlight: true
    value_overlay: true
    haptic: light_impact (iOS)
```

### Inspector Panel (Adaptive)
```yaml
structure:
  header:
    title: "Inspector"
    stats: entity_count
    close_button: trailing
    
  outliner:
    style: hierarchical_list
    icons: sf_symbols
    selection: follows_scene
    
  properties:
    style: grouped_form
    resize_handle: top_edge
    min_height: 100pt
    max_height: 400pt
    
responsive:
  regular:
    width: 320pt
    position: NavigationSplitView.sidebar
    
  compact:
    width: 80%_screen
    position: sheet_overlay
    drag_to_dismiss: true
```

## Platform-Specific Refinements

### macOS
```yaml
window:
  min_size: 800x600
  style: standard_window
  
interactions:
  hover_effects: enabled
  context_menus: right_click
  keyboard_shortcuts: visible
  cursor_changes: true
  
toolbar:
  style: unified_compact
  customizable: false
```

### iOS/iPadOS
```yaml
safe_areas: respected
gestures:
  pinch_zoom: viewport_camera
  two_finger_drag: pan_camera
  long_press: context_menu
  
haptics:
  selection: light_impact
  mode_change: selection_changed
  error: notification_error
  
presentation:
  inspector: slide_over
  sheets: medium_large_detents
```

### tvOS
```yaml
focus_engine: enabled
controls:
  remote_swipe: camera_orbit
  play_pause: toggle_selection
  menu: back_navigation
  
simplifications:
  toolbar: essential_only
  inspector: read_only
  gizmos: larger_targets
```

## Visual Effects & Materials

### SwiftUI Materials
```yaml
panels:
  primary: .regularMaterial
  secondary: .thinMaterial
  overlay: .ultraThinMaterial
  
backgrounds:
  viewport: Color.clear  # Shows Metal layers
  inspector: .grouped
  toolbar: .regularMaterial
```

### Metal Shader Effects
```yaml
sky_gradient:
  type: vertex_fragment_shader
  blend: smooth_interpolation
  update: per_frame
  
grid_fade:
  type: fragment_shader
  function: smoothstep(5.0, 20.0, distance)
  alpha: distance_based
  
transparency:
  order: back_to_front
  blend: source_over
```

## Animation Specifications

### Standard Transitions
```yaml
inspector_toggle:
  duration: 0.3s
  curve: spring(response: 0.3)
  
selection_change:
  duration: 0.2s
  curve: easeInOut
  
gizmo_hover:
  duration: 0.15s
  scale: 1.1x
  
mode_switch:
  duration: 0.25s
  type: opacity_transition
```

### Day/Night Cycle
```yaml
phase_transition:
  duration: continuous
  interpolation: linear
  update_rate: 60fps
  total_cycle: 120s
```

## Typography & Icons

### Text Hierarchy
| Level | SwiftUI Style | Size | Weight | Usage |
|-------|--------------|------|--------|-------|
| Title | .headline | System | .medium | Panel headers |
| Body | .body | System | .regular | General text |
| Label | .caption | System | .regular | Property labels |
| Value | .body | System | .monospaced | Numbers |

### SF Symbols Usage
| Category | Symbol | Size | Rendering |
|----------|--------|------|-----------|
| Camera | camera.fill | 20pt | hierarchical |
| Light | light.max | 20pt | multicolor |
| Model | cube.fill | 20pt | monochrome |
| Transform | move.3d | 16pt | monochrome |

## Accessibility

### Dynamic Type
```yaml
supported: true
minimum_size: .small
maximum_size: .xxxLarge
layout_adjustments: automatic
```

### High Contrast
```yaml
grid_opacity: increased to 0.6
selection_outline: 2pt width
gizmo_colors: high_contrast_variants
```

### Reduce Motion
```yaml
transitions: cross_fade_only
animations: disabled
day_night_cycle: manual_control
```

## Performance Visual Targets

### Frame Rate
```yaml
target: 60fps
minimum: 30fps
measurement: Metal HUD

budget:
  sky: 0.5ms
  grid: 0.3ms
  entities: 8-12ms
  ui: 2ms
  total: <16.67ms
```

### Visual Quality Settings
```yaml
high_quality:
  grid_divisions: 20
  shadow_resolution: 2048
  antialiasing: 4x
  
balanced:
  grid_divisions: 10
  shadow_resolution: 1024
  antialiasing: 2x
  
performance:
  grid_divisions: 5
  shadow_resolution: 512
  antialiasing: none
```

## Visual Debug Overlays

### Debug Mode
```yaml
fps_counter:
  position: top_right
  style: monospaced
  
entity_bounds:
  color: yellow
  style: wireframe
  
selection_info:
  shows: name, type, transform
  position: above_gizmo
  
performance_hud:
  shows: draw_calls, triangles, frame_time
  position: bottom_right
```

## Summary

The visual system combines:
- ✅ **Metal rendering** for GPU-accelerated backgrounds and grids
- ✅ **Adaptive layouts** following WWDC25 best practices
- ✅ **Platform optimizations** for macOS, iOS, and tvOS
- ✅ **Dynamic theming** with day/night cycle
- ✅ **Consistent design language** across all platforms

This unified visual system ensures professional appearance while maintaining 60fps performance across all supported devices.