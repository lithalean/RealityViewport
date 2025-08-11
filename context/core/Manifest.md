# RealityViewport Context Engineering Manifest

**Purpose**: Master reference for AI systems to understand the context module structure  
**Last Updated**: August 2025  
**Manifest Version**: 3.1  
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
| **ARCHITECTURE.md** | 3.0 | ✅ CURRENT | August 2025 | COMPLETE | Entity/ECS architecture documented |
| **IMPLEMENTATION.md** | 3.0 | ✅ CURRENT | August 2025 | COMPLETE | Current state with all managers |
| **ENTITY_SYSTEM.md** | 1.0 | ✅ CURRENT | August 2025 | COMPLETE | Full Entity/ECS documentation |
| **METAL_RENDERING.md** | 1.0 | ✅ CURRENT | August 2025 | COMPLETE | Metal pipeline documentation |
| **VISUAL.md** | 3.0 | ✅ CURRENT | August 2025 | COMPLETE | Includes adaptive UI patterns |
| **EVOLUTION.md** | 3.0 | ✅ CURRENT | August 2025 | COMPLETE | Migration journey documented |
| **NAVIGATION.md** | 2.0 | 🔴 OUTDATED | July 2025 | MEDIUM | Still references Node system |
| **VIEWPORTSTATE.md** | 1.0 | 🔴 OUTDATED | July 2025 | MEDIUM | Pre-Entity system docs |
| **FILEOPERATIONS.md** | 1.0 | 🔴 OUTDATED | July 2025 | LOW | Needs Entity updates |
| **GESTURES.md** | 1.0 | 🔴 OUTDATED | July 2025 | LOW | Still mostly accurate |
| **INTEGRATION.yaml** | 1.0 | 🔴 OUTDATED | July 2025 | LOW | Dependencies changed |
| **session.json** | - | ⚠️ DYNAMIC | Runtime | - | Not updated for Entity system |

## Documentation Update Status

### ✅ Fully Updated (v3.0)
- **ARCHITECTURE.md** - Complete Entity/ECS rewrite
- **IMPLEMENTATION.md** - All managers and current state
- **ENTITY_SYSTEM.md** - New comprehensive guide
- **METAL_RENDERING.md** - New GPU pipeline docs
- **VISUAL.md** - Adaptive UI integrated
- **EVOLUTION.md** - Full migration story

### 🔴 Not Yet Updated (Still v2.0 or older)
- **NAVIGATION.md** - Still references Nodes, needs Entity updates
- **VIEWPORTSTATE.md** - Pre-migration, needs RealityKit.Entity docs
- **FILEOPERATIONS.md** - Works but references old Node system
- **GESTURES.md** - Mostly accurate but needs Entity context
- **INTEGRATION.yaml** - Dependencies and external connections outdated
- **session.json** - Runtime state structure needs Entity format

## Quick Reference Matrix v3.1

| Information Needed | Primary Module | Secondary Module | Notes |
|-------------------|----------------|------------------|-------|
| **Entity System** | ENTITY_SYSTEM.md | ARCHITECTURE.md | ✅ Current |
| **Metal Rendering** | METAL_RENDERING.md | VISUAL.md | ✅ Current |
| **How something works** | ARCHITECTURE.md | IMPLEMENTATION.md | ✅ Current |
| **Current state** | IMPLEMENTATION.md | - | ✅ Current |
| **UI/Visual design** | VISUAL.md | - | ✅ Current |
| **Migration history** | EVOLUTION.md | - | ✅ Current |
| **Navigation flow** | NAVIGATION.md | - | 🔴 OUTDATED - Node refs |
| **Viewport state** | VIEWPORTSTATE.md | ENTITY_SYSTEM.md | 🔴 OUTDATED - needs update |
| **File operations** | FILEOPERATIONS.md | IMPLEMENTATION.md | 🔴 OUTDATED - Node refs |
| **Input handling** | GESTURES.md | - | 🔴 OUTDATED - minor |
| **External deps** | INTEGRATION.yaml | - | 🔴 OUTDATED |

## Context Loading Strategy v3.1

### For Bug Fixes (Current Architecture)
1. **ENTITY_SYSTEM.md** - Understand entity architecture ✅
2. **IMPLEMENTATION.md** - Current manager states ✅
3. **METAL_RENDERING.md** - If rendering-related ✅
4. **ARCHITECTURE.md** - Overall patterns ✅

### For Understanding Old References
If you encounter Node references in outdated docs:
1. Know that Node system is COMPLETELY REMOVED
2. Refer to ENTITY_SYSTEM.md for current approach
3. Check EVOLUTION.md for migration details

### ⚠️ WARNING: Outdated Modules
The following modules contain outdated Node system references and should be cross-referenced with updated docs:
- NAVIGATION.md
- VIEWPORTSTATE.md  
- FILEOPERATIONS.md
- GESTURES.md
- INTEGRATION.yaml
- session.json

## Project Statistics (August 2025)

### Codebase Metrics
- **Total Swift Files**: 60
- **Total Metal Files**: 2
- **Lines of Code**: ~12,000
- **Architecture**: Entity/ECS Hybrid
- **Completion**: ~70%

### Documentation Coverage
- **Updated for v3.0**: 6/12 modules (50%)
- **Outdated (v2.0)**: 6/12 modules (50%)
- **Critical docs**: 100% updated
- **Secondary docs**: 0% updated

## Critical Components v3.1

### Entity System (CURRENT)
- **Entity**: Base wrapper around RealityKit.Entity
- **SceneEntity Protocol**: Interface for all entity types
- **CameraEntity**: Perspective camera management
- **LightEntity**: Dynamic lighting
- **ModelEntity**: 3D model loading

### Metal Rendering (CURRENT)
- **MetalRenderer**: Core Metal pipeline
- **MetalGridRenderer**: GPU-accelerated grid
- **MetalSkyRenderer**: Day/night cycle
- **Shaders**: Custom Metal shaders

### Managers (CURRENT)
- **SceneManager**: Entity lifecycle
- **SelectionManager**: Multi-selection with gizmos
- **ProjectManager**: Project save/load
- **ControlManager**: Input handling
- **DayNightManager**: Atmospheric control

### ViewportState (NEEDS UPDATE)
- Uses `RealityKit.Entity` directly (not custom Entity)
- Documentation outdated but code is current
- See ENTITY_SYSTEM.md for actual implementation

## Documentation Debt

### High Priority Updates Needed
1. **VIEWPORTSTATE.md** - Critical for understanding RealityKit.Entity usage
2. **NAVIGATION.md** - User flows need Entity context

### Medium Priority Updates
3. **FILEOPERATIONS.md** - Update for Entity serialization
4. **INTEGRATION.yaml** - Update dependencies

### Low Priority Updates
5. **GESTURES.md** - Minor Entity references
6. **session.json** - Runtime state format

## AI Assistant Guidelines

### When Using This Documentation
1. **Always check module status** before trusting content
2. **Prefer v3.0 modules** for current information
3. **Cross-reference outdated modules** with ENTITY_SYSTEM.md
4. **Assume Node system doesn't exist** - it's completely removed

### Critical Rem# RealityViewport Context Engineering Manifest

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

### When Using This Documentation
1. **Always check module status** before trusting content
2. **Prefer v3.0 modules** for current information
3. **Cross-reference outdated modules** with ENTITY_SYSTEM.md
4. **Assume Node system doesn't exist** - it's completely removed

### Critical Reminders
- **Entity vs RealityKit.Entity** - Always disambiguate
- **Node system is GONE** - Any Node reference is outdated
- **ViewportState uses RealityKit.Entity** - Not custom Entity
- **Adaptive UI is standard** - Not a special feature

## Module Dependencies

```
Updated Modules (v3.0):
ENTITY_SYSTEM.md
    ├── ARCHITECTURE.md ✅
    ├── IMPLEMENTATION.md ✅
    └── METAL_RENDERING.md ✅
    
VISUAL.md ✅
    └── (Includes adaptive UI)
    
EVOLUTION.md ✅
    └── (Complete migration story)

Outdated Modules (v2.0 or older):
NAVIGATION.md 🔴
    └── Needs Entity updates
    
VIEWPORTSTATE.md 🔴
    └── Needs RealityKit.Entity docs
    
FILEOPERATIONS.md 🔴
    └── Needs Entity serialization
    
GESTURES.md 🔴
    └── Minor updates needed
    
INTEGRATION.yaml 🔴
    └── Dependencies outdated
    
session.json 🔴
    └── Runtime format outdated
```

## Documentation Update Plan

### Phase 1: Critical Updates (Immediate)
- [ ] Update VIEWPORTSTATE.md for RealityKit.Entity usage
- [ ] Update NAVIGATION.md for Entity-based flows

### Phase 2: Secondary Updates (Next Sprint)
- [ ] Update FILEOPERATIONS.md for Entity serialization
- [ ] Update INTEGRATION.yaml with current dependencies
- [ ] Update session.json format

### Phase 3: Minor Updates (When Convenient)
- [ ] Update GESTURES.md for Entity interactions
- [ ] Add unit test documentation
- [ ] Add performance profiling guide

## Summary

**Current State**: 
- Core documentation (50%) fully updated for Entity/ECS architecture
- Secondary documentation (50%) still contains Node references
- Code is 100% migrated, docs are 50% migrated

**Critical Note for AI Systems**:
When encountering any reference to "Node", "SceneNode", "BaseSceneNode", "CameraNode", "LightNode", or "ModelNode" in documentation, that documentation is OUTDATED. The current system uses Entity, CameraEntity, LightEntity, and ModelEntity exclusively.

---

**⚠️ IMPORTANT**: This manifest reflects a system in transition. While the code is fully migrated to Entity/ECS + Metal, half the documentation still references the old Node system. Always verify information against the updated v3.0 modules when in doubt.