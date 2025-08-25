# RealityViewport Visual Design System

**Purpose**: Complete visual design system, UI specifications, and floating UI layouts  
**Version**: 4.0  
**Design Language**: Apple HIG + Metal Rendering + Floating Glass UI  
**Last Updated**: August 25 2025

## Visual Architecture
```
RENDERING LAYERS (back→front):
├─[0] Metal Sky (Dynamic gradient background)
├─[1] Metal Grid (GPU-accelerated reference grid)
├─[2] RealityKit Scene (3D models & entities)
├─[3] Billboard Icons (Camera-facing UI elements)
├─[4] Transform Gizmos (Manipulation tools)
├─[5] Viewport Overlays (Axis helper, stats)
└─[6] Floating UI Layer (Glass panels, edge-to-edge)
```

## Floating UI Design System

### Core Principles
```yaml
philosophy:
  - Edge-to-edge viewport maximizes 3D space
  - Floating glass panels preserve context
  - Consistent styling across all platforms
  - No navigation chrome or wasted space

visual_language:
  material: .ultraThinMaterial
  corner_radius: 12pt
  shadow: color: .black.opacity(0.2), radius: 10
  padding: 20pt from screen edges
  animation: .easeInOut(duration: 0.25)
```

### Layout Architecture
```swift
// Single ZStack for all platforms (simplified from v3.0)
ZStack {
    // Full-screen viewport
    ViewportStack
        .ignoresSafeArea()
    
    // Floating overlays
    VStack {
        // Top toolbar
        ViewportToolbar()
            .floatingPanel()
        Spacer()
        // Bottom status
        StatusBar()
            .floatingPanel()
    }
    
    HStack {
        Spacer()
        // Right inspector
        if showInspector {
            Inspector()
                .floatingPanel()
        }
    }
}
```

### Floating Panel Specifications
```yaml
standard_panel:
  background: .ultraThinMaterial
  corner_radius: 12pt
  shadow:
    color: .black.opacity(0.2)
    radius: 10pt
    x: 0 (toolbar/status), -2 (inspector)
    y: 2 (toolbar), -2 (status), 0 (inspector)
  
toolbar:
  position: top center
  width: ~85% of screen
  height: 44pt
  padding: horizontal: 20pt, top: 12pt
  content: centered horizontally
  
inspector:
  position: right edge
  width: 320pt (macOS), 280pt (iOS)
  height: flexible (max screen - 100pt)
  padding: right: 20pt, vertical: 70pt
  animation: slide + fade from right
  
status_bar:
  position: bottom center
  width: ~85% of screen
  height: auto (content-sized)
  padding: horizontal: 20pt, bottom: 12pt
  content: project info left, stats right
```

## Visual Hierarchy Update

### Before (v3.0) vs After (v4.0)
```yaml
Before (Complex Adaptive):
  - NavigationSplitView for iPad/Mac
  - NavigationStack for iPhone
  - Different layouts per platform
  - Inspector in sidebar (left)
  - Toolbar embedded in navigation
  - Multiple code paths
  
After (Unified Floating):
  - Single ZStack architecture
  - Same layout all platforms
  - Edge-to-edge viewport
  - Floating inspector (right)
  - Floating toolbar (top)
  - One code path
  
Visual Benefits:
  - More viewport space (+15-20%)
  - Consistent experience
  - Modern glass aesthetic
  - Better depth perception
  - Cleaner visual hierarchy
```

## Updated Color System

### Glass Material Palette
| Element | Material | Opacity | Usage |
|---------|----------|---------|--------|
| **Toolbar** | .ultraThinMaterial | System | Top floating bar |
| **Inspector** | .regularMaterial | System | Right panel background |
| **Status** | .ultraThinMaterial | System | Bottom info bar |
| **Viewport** | Color.clear | 100% | Full transparency |

### Core Palette (Unchanged)
| Element | Light Mode | Dark Mode | Metal/Hex | Usage |
|---------|------------|-----------|-----------|--------|
| **Sky** | Dynamic | Dynamic | See cycle | Background gradient |
| **Grid** | 0.3 alpha gray | 0.3 alpha gray | #4C4C4C80 | Reference grid |
| **Selection** | accentColor | accentColor | System | Selected objects |
| **Gizmo X** | .red | .red | #FF0000 | X-axis |
| **Gizmo Y** | .green | .green | #00FF00 | Y-axis |
| **Gizmo Z** | .blue | .blue | #0000FF | Z-axis |

## Component Specifications

### Floating Viewport Toolbar
```yaml
container:
  type: floating_glass_bar
  position: top center
  width: fill - 40pt (20pt each side)
  height: 44pt
  
visual_style:
  background: .ultraThinMaterial
  corner_radius: 12pt
  shadow: (0, 2, 10, 0.2)
  border: none
  
layout:
  alignment: horizontal center
  spacing: 10pt between groups
  dividers: 20pt height, 1pt width
  padding: 6pt internal
  
controls:
  project_management:
    [folder, save, export]
    size: 26pt buttons
    
  entity_creation:
    [+ menu]
    size: 26pt button
    
  interaction_mode:
    [environment, entity]
    style: segmented
    background: .gray.opacity(0.1)
    
  camera_mode:
    [orbit, fly, pan]
    style: segmented
    
  inspector_toggle:
    [sidebar.right]
    position: trailing
```

### Floating Inspector Panel
```yaml
container:
  type: floating_glass_panel
  position: right edge
  width: 320pt (macOS), 280pt (iOS)
  max_height: screen - 100pt
  
visual_style:
  background: .regularMaterial
  corner_radius: 12pt
  shadow: (-2, 0, 10, 0.2)
  
animation:
  show: slide(from: .trailing) + opacity
  hide: slide(to: .trailing) + opacity
  duration: 0.25s
  curve: .easeInOut
  
structure:
  header:
    tabs: [Outliner, Properties]
    close_button: trailing X
    height: 36pt
    
  content:
    outliner:
      style: hierarchical_list
      icons: sf_symbols
      
    properties:
      style: grouped_form
      resizable: true
      resize_handle: top edge
```

### Floating Status Bar
```yaml
container:
  type: floating_glass_bar
  position: bottom center
  width: fill - 40pt
  height: auto (28-32pt typical)
  
visual_style:
  background: .ultraThinMaterial
  corner_radius: 12pt
  shadow: (0, -2, 10, 0.2)
  
content:
  left:
    project_name: .caption
    modified_indicator: if changed
    
  right:
    statistics:
      - entity_count
      - light_count
      - camera_count
    style: .caption2
    color: .secondary
```

## Platform-Specific Refinements

### macOS
```yaml
floating_ui:
  inspector_width: 320pt
  toolbar_height: 44pt
  button_size: 26pt
  
interactions:
  hover_effects: enabled
  cursor_changes: true
  trackpad_gestures: pinch zoom
  
performance:
  update_strategy: on_demand
  uses_needsUpdate: true
```

### iOS/iPadOS
```yaml
floating_ui:
  inspector_width: 280pt
  toolbar_height: 44pt
  button_size: 32pt (compact)
  
interactions:
  touch_gestures: all enabled
  haptic_feedback: light/medium
  safe_area_insets: respected
  
performance:
  update_strategy: timer_based
  timer_rate: 60fps
  avoids_publishing_conflicts: true
```

### tvOS
```yaml
floating_ui:
  simplified: true
  inspector: read_only
  focus_indicators: enhanced
  
adaptations:
  toolbar: essential_only
  no_floating_panels: true
  control_hints: visible
```

## Visual Effects & Glass Morphism

### Material Hierarchy
```yaml
depth_levels:
  0: viewport (transparent)
  1: floating_panels (.ultraThinMaterial)
  2: inspector_content (.regularMaterial)
  3: modal_sheets (.thickMaterial)
  
glass_effects:
  blur: system_managed
  vibrancy: automatic
  transparency: follows_system
  
shadows:
  standard: (0, 2, 10, 0.2)
  inspector: (-2, 0, 10, 0.2)
  status: (0, -2, 10, 0.2)
```

### Consistent Styling
```swift
// Reusable floating panel modifier
extension View {
    func floatingPanel() -> some View {
        self
            .background(.ultraThinMaterial)
            .clipShape(RoundedRectangle(cornerRadius: 12))
            .shadow(color: .black.opacity(0.2), radius: 10)
    }
}
```

## Animation Specifications

### Floating UI Transitions
```yaml
inspector_toggle:
  type: slide + opacity
  duration: 0.25s
  curve: .easeInOut
  direction: horizontal (from right)
  
toolbar_appearance:
  type: opacity + scale
  duration: 0.3s
  initial_scale: 0.95
  
panel_resize:
  type: continuous
  constraint: maintain aspect
  
hover_states:
  scale: 1.02
  duration: 0.15s
  buttons_only: true
```

### Day/Night Cycle (Unchanged)
```yaml
phase_transition:
  duration: continuous
  interpolation: linear
  update_rate: 60fps
  total_cycle: 120s
```

## Typography & Icons

### Floating UI Text Styles
| Element | Style | Size | Weight | Color |
|---------|-------|------|--------|-------|
| Toolbar | .body | 13pt | .medium | .primary |
| Inspector Title | .headline | 13pt | .medium | .primary |
| Inspector Tab | .body | 13pt | .regular | .secondary |
| Status Info | .caption | 11pt | .regular | .secondary |
| Button Label | .caption2 | 10pt | .regular | .secondary |

### Icon Specifications
```yaml
toolbar_icons:
  size: 16pt (regular), 20pt (compact)
  weight: .medium
  rendering: monochrome
  
inspector_icons:
  size: 14pt
  weight: .regular
  rendering: hierarchical
  
status_icons:
  size: 12pt
  weight: .regular
  color: .secondary
```

## Accessibility

### Floating UI Adaptations
```yaml
reduce_transparency:
  panels: .regularMaterial (more opaque)
  maintains_hierarchy: true
  
increase_contrast:
  panel_borders: 1pt .separator
  shadow_intensity: increased
  
voice_over:
  panel_descriptions: complete
  navigation_hints: provided
  
dynamic_type:
  panels_resize: true
  minimum_width: adjusted
  scroll_enabled: when needed
```

## Performance Visual Targets

### Frame Rate with Floating UI
```yaml
target: 60fps
minimum: 30fps

budget:
  sky: 0.5ms
  grid: 0.3ms
  entities: 8-12ms
  floating_ui: 0.5ms (negligible)
  swiftui: 2ms
  total: <16.67ms
  
iOS_specific:
  timer_updates: 60fps consistent
  no_publishing_conflicts: true
  smooth_animations: true
```

### Visual Quality Settings
```yaml
high_quality:
  panel_blur: full
  shadows: all enabled
  animations: full
  
balanced:
  panel_blur: reduced
  shadows: simplified
  animations: reduced
  
performance:
  panel_blur: minimal
  shadows: none
  animations: cross_fade_only
```

## Visual Debug Overlays

### Debug Mode with Floating UI
```yaml
layout_guides:
  shows: panel boundaries
  color: .purple
  style: dashed lines
  
touch_indicators:
  shows: touch points
  style: circles
  fade_duration: 0.5s
  
performance_overlay:
  position: top_right (below toolbar)
  shows: fps, frame_time
  background: .black.opacity(0.5)
```

## Design Rationale

### Why Floating UI?
```yaml
benefits:
  - Maximum viewport space
  - Context preservation (see through panels)
  - Modern aesthetic (iOS 16+ style)
  - Consistent cross-platform
  - Simpler codebase
  
tradeoffs:
  - No docking (not needed)
  - No multi-window (YAGNI)
  - Fixed panel sizes (intentional)
```

## Summary

The visual system v4.0 delivers:
- ✅ **Edge-to-edge viewport** maximizing 3D workspace
- ✅ **Floating glass panels** with consistent styling
- ✅ **Unified design** across all platforms
- ✅ **Modern aesthetic** following latest Apple HIG
- ✅ **60fps performance** maintained with UI overlays
- ✅ **Simplified codebase** with single layout path

This floating UI system provides a professional, modern appearance while maintaining the simplicity and performance requirements of the RealityViewport alpha.