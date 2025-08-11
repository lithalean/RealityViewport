# RealityViewport Context Engineering Manifest

**Purpose**: Master reference for AI systems to understand the context module structure  
**Last Updated**: August 2025  
**Manifest Version**: 3.0  
**Architecture**: Entity/ECS + Metal Rendering + Adaptive UI

## 🚨 CRITICAL ARCHITECTURE CHANGES (v3.0)

### Removed Systems
- ❌ **Node System** - Completely removed (Core/Nodes/)
- ❌ **SceneNode Protocol** - Replaced with SceneEntity
- ❌ **Platform-Specific Views** - No more iPhoneView/MacView

### New Systems
- ✅ **Entity/ECS Architecture** - Unity DOTS-inspired hybrid system
- ✅ **Metal Rendering Pipeline** - GPU-accelerated viewport
- ✅ **Adaptive UI** - Single ContentView for all platforms
- ✅ **Day/Night System** - Dynamic atmospheric rendering

## Module Version Registry

| Module | Version | Status | Last Modified | Priority | Description |
|--------|---------|--------|---------------|----------|-------------|
| ARCHITECTURE.md | 3.0 | 🔴 OUTDATED | August 2025 | CRITICAL | Must reflect Entity/ECS system |
| IMPLEMENTATION.md | 3.0 | 🔴 OUTDATED | August 2025 | CRITICAL | Must document new managers |
| ENTITY_SYSTEM.md | 1.0 | 🆕 NEW | August 2025 | CRITICAL | Documents Entity/ECS architecture |
| METAL_RENDERING.md | 1.0 | 🆕 NEW | August 2025 | HIGH | Metal pipeline documentation |
| ADAPTIVE_UI.md | 1.0 | 🆕 NEW | August 2025 | HIGH | Unified UI documentation |
| VISUAL.md | 3.0 | 🟡 UPDATE | August 2025 | MEDIUM | Add Metal rendering info |
| NAVIGATION.md | 3.0 | 🟡 UPDATE | August 2025 | MEDIUM | Update for adaptive UI |
| INTEGRATION.yaml | 2.0 | 🟡 UPDATE | August 2025 | LOW | Add new dependencies |
| EVOLUTION.md | 3.0 | 🔴 OUTDATED | August 2025 | HIGH | Document migration |
| VIEWPORTSTATE.md | 2.0 | 🟡 UPDATE | August 2025 | MEDIUM | RealityKit.Entity usage |
| FILEOPERATIONS.md | 2.0 | ✅ CURRENT | August 2025 | LOW | Minor updates only |
| GESTURES.md | 2.0 | ✅ CURRENT | August 2025 | LOW | Still accurate |
| session.json | - | DYNAMIC | - | - | Runtime state |

## Quick Reference Matrix v3.0

| Information Needed | Primary Module | Secondary Module | Notes |
|-------------------|----------------|------------------|-------|
| **Entity System** | ENTITY_SYSTEM.md | ARCHITECTURE.md | NEW - Critical for understanding |
| **Metal Rendering** | METAL_RENDERING.md | IMPLEMENTATION.md | NEW - GPU pipeline |
| **How something works** | ARCHITECTURE.md | ENTITY_SYSTEM.md | Architecture completely changed |
| **Current managers** | IMPLEMENTATION.md | - | 5 managers + Metal renderers |
| **UI/Layout** | ADAPTIVE_UI.md | NAVIGATION.md | Unified adaptive system |
| **Visual rendering** | METAL_RENDERING.md | VISUAL.md | Metal shaders + day/night |
| **Component system** | ENTITY_SYSTEM.md | ARCHITECTURE.md | ECS components |
| **Platform differences** | ADAPTIVE_UI.md | IMPLEMENTATION.md | Single adaptive view |
| **Migration history** | EVOLUTION.md | - | Node→Entity transition |
| **File operations** | FILEOPERATIONS.md | IMPLEMENTATION.md | Updated for entities |

## Context Loading Strategy v3.0

### For Bug Fixes
1. **ENTITY_SYSTEM.md** - Understand entity architecture
2. **IMPLEMENTATION.md** - Current manager states
3. **METAL_RENDERING.md** - If rendering-related
4. **session.json** - Recent runtime changes

### For New Features
1. **ARCHITECTURE.md** - Overall design patterns
2. **ENTITY_SYSTEM.md** - Entity/component patterns
3. **ADAPTIVE_UI.md** - UI integration points
4. **NAVIGATION.md** - User flow
5. **METAL_RENDERING.md** - If visual features

### For Entity Operations
1. **ENTITY_SYSTEM.md** - Entity lifecycle
2. **IMPLEMENTATION.md** - SceneManager operations
3. **ARCHITECTURE.md** - Component patterns

### For Rendering Issues
1. **METAL_RENDERING.md** - Metal pipeline
2. **VIEWPORTSTATE.md** - Viewport management
3. **IMPLEMENTATION.md** - Renderer classes

## Project Statistics (August 2025)

### Codebase Metrics
- **Total Swift Files**: 60
- **Total Metal Files**: 2
- **Lines of Code**: ~12,000
- **Architecture**: Entity/ECS Hybrid
- **Completion**: ~70%

### Core Systems
- **Entities**: 5 types (Entity, Camera, Light, Model, Scene)
- **Managers**: 5 (Scene, Selection, Project, Control, DayNight)
- **Renderers**: 3 (Metal, MetalGrid, MetalSky)
- **Shaders**: 2 (Grid, DayNightBackground)

### Technology Stack
- **UI Framework**: SwiftUI (100% pure)
- **3D Engine**: RealityKit
- **Rendering**: Metal + RealityKit
- **Platforms**: iOS, macOS, tvOS
- **Min Deployment**: iOS 17.0, macOS 14.0

## Critical Components v3.0

### Entity System (NEW)
- **Entity**: Base wrapper around RealityKit.Entity
- **SceneEntity Protocol**: Interface for all entity types
- **CameraEntity**: Perspective camera management
- **LightEntity**: Dynamic lighting (point, spot, directional)
- **ModelEntity**: 3D model loading and management

### Metal Rendering (NEW)
- **MetalRenderer**: Core Metal pipeline
- **MetalGridRenderer**: GPU-accelerated grid
- **MetalSkyRenderer**: Day/night cycle rendering
- **Shaders**: Custom Metal shaders for effects

### Managers
- **SceneManager**: Entity lifecycle and scene graph
- **SelectionManager**: Multi-selection with gizmos
- **ProjectManager**: Project save/load with new format
- **ControlManager**: Input handling across platforms
- **DayNightManager**: Atmospheric lighting control

### Adaptive UI (NEW)
- **ContentView**: Single adaptive layout
- **NavigationSplitView**: Regular layouts (iPad/Mac)
- **NavigationStack**: Compact layouts (iPhone)
- **Inspector**: Unified property editing

### Viewport System
- **ViewportState**: RealityKit.Entity management
- **ViewportView**: Main 3D viewport
- **ViewportMetalGrid**: Metal-rendered grid
- **TransformGizmo**: Visual manipulation tools

## Migration Notes (v2.0 → v3.0)

### Breaking Changes
1. All Node references must be updated to Entity
2. SceneNode protocol → SceneEntity protocol
3. Platform-specific views removed
4. ViewportState uses RealityKit.Entity directly

### New Capabilities
1. GPU-accelerated rendering via Metal
2. Dynamic day/night cycle
3. Improved performance (60fps target)
4. Unified adaptive UI
5. Better component composition

### Known Issues
- SpotLight not fully implemented (using PointLight)
- Some Entity properties placeholder
- ~30% features still pending

## AI Assistant Guidelines

### When Working with Entities
- Always distinguish between custom `Entity` and `RealityKit.Entity`
- Use explicit namespacing when ambiguous
- Entity system is Unity DOTS-inspired but Apple-native

### When Working with UI
- Single ContentView adapts to all platforms
- Use NavigationSplitView for regular layouts
- Use NavigationStack for compact layouts
- No platform-specific view files

### When Working with Rendering
- Metal pipeline handles grid and sky
- RealityKit handles 3D models
- ViewportState manages RealityKit entities
- Day/night cycle is automatic

## Module Dependencies

```
ENTITY_SYSTEM.md
    ├── ARCHITECTURE.md
    ├── IMPLEMENTATION.md
    └── METAL_RENDERING.md
    
ADAPTIVE_UI.md
    ├── NAVIGATION.md
    └── VISUAL.md
    
METAL_RENDERING.md
    ├── VIEWPORTSTATE.md
    └── VISUAL.md
    
IMPLEMENTATION.md
    ├── ENTITY_SYSTEM.md
    ├── METAL_RENDERING.md
    └── ADAPTIVE_UI.md
```

## Next Steps for Documentation Update

1. ✅ Update Manifest.md (THIS FILE)
2. 🔴 Create ENTITY_SYSTEM.md
3. 🔴 Create METAL_RENDERING.md
4. 🔴 Create ADAPTIVE_UI.md
5. 🔴 Update ARCHITECTURE.md
6. 🔴 Update IMPLEMENTATION.md
7. 🔴 Update EVOLUTION.md
8. 🟡 Update remaining modules

---

**⚠️ CRITICAL**: This project has undergone a complete architectural transformation. All v2.0 documentation is obsolete. Prioritize understanding the Entity/ECS system and Metal rendering pipeline.