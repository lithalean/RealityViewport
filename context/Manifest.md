# RealityViewport Context Engineering Manifest

**Module**: MANIFEST.md  
**Version**: 7.0  
**Philosophy**: Apple-native Unity DOTS alternative  
**Architecture**: Simplified Entity Wrapper + Metal Rendering + Floating UI  
**Status**: Phase 1 - 98% Complete (Grid Sync Remaining)  
**Last Updated**: December 2024

## 🎯 Project Philosophy

**"What if Apple made Unity with DOTS?"**

RealityViewport is an **Apple-native 3D editor** that combines:
- **Simplified Entity wrappers** that grow with your needs
- **RealityKit's powerful ECS** as the foundation
- **Metal GPU rendering** for atmospheric effects
- **Beautiful floating glass UI** maximizing viewport space
- **Immediate primitive visibility** with colored materials ✅

This is not about building everything on day one. It's about shipping a working alpha with a solid foundation that can evolve into whatever your game needs.

## 🚀 Phase 1 Status: 98% Complete

### What's Complete (98%)
- ✅ **Primitive Visibility Fixed** - Colored primitives appear immediately
- ✅ **Scene Graph Connected** - ViewportState.setSceneManager() implemented
- ✅ **Platform Compatibility** - iOS/macOS working with proper color types
- ✅ **Publishing Errors Fixed** - Task deferral pattern working
- ✅ **60fps Performance** - Maintained with headroom

### What Remains (2%)
- ⚠️ **Grid Synchronization** - Minor lag during camera movement
  - Reduced from 16ms to <1ms but not frame-perfect
  - Occasional desync on rapid movements
  - Required for Phase 1 completion

## 📚 Document Reading Guide

### For Different Purposes

#### 🎯 "I want to understand the system"
1. **Architecture.md** - Design philosophy and patterns
2. **EntitySystem.md v3.0** - Simplified Entity wrapper with primitives
3. **ViewportState.md v5.0** - Scene graph connection discovery
4. **Implementation.md v5.0** - Phase 1 completion status

#### 🐛 "I need to fix a bug"
1. **MetalRendering.md v2.0** - Grid sync issues (SYNC-001/002/003)
2. **Implementation.md v5.0** - Known issues and fixes
3. Component's specific manager documentation

#### 🚀 "I want to add a feature"
1. **Architecture.md** - Patterns to follow (especially YAGNI)
2. **EntitySystem.md v3.0** - How primitives were added
3. **Visual.md** - UI integration patterns
4. **Navigation.md** - User flow integration

#### 📁 "I'm working with files"
1. **FileOperations.md** - Serialization patterns
2. **Navigation.md** - Import/export flows
3. **EntitySystem.md** - Entity data structures

## Module Version Registry

| Module | Version | Status | Last Updated | Key Changes |
|--------|---------|--------|--------------|-------------|
| **IMPLEMENTATION.md** | 5.0 | ✅ PHASE 1 | December 2024 | Primitive visibility complete |
| **ENTITY_SYSTEM.md** | 3.0 | ✅ PHASE 1 | December 2024 | Colored primitive factories |
| **VIEWPORTSTATE.md** | 5.0 | ✅ PHASE 1 | December 2024 | setSceneManager() connection |
| **METAL_RENDERING.md** | 2.0 | ⚠️ 98% DONE | December 2024 | Grid sync issues documented |
| **ARCHITECTURE.md** | 3.0 | ✅ CURRENT | August 2025 | Apple-native Unity DOTS philosophy |
| **NAVIGATION.md** | 5.0 | ✅ CURRENT | August 2025 | Floating UI flows |
| **VISUAL.md** | 4.0 | ✅ CURRENT | August 2025 | Glass morphism design |
| **GESTURES.md** | 3.1 | ✅ CURRENT | August 2025 | iOS fixes documented |
| **EVOLUTION.md** | 4.0 | 📝 NEEDS UPDATE | August 2025 | Missing Phase 1 changes |
| **INTEGRATION.yaml** | 3.0 | 📝 NEEDS UPDATE | August 2025 | Missing Phase 1 integrations |
| **FILEOPERATIONS.md** | 2.0 | ✅ ALIGNED | August 2025 | Entity serialization |
| **session.json** | 4.0 | DYNAMIC | Runtime | Current state |

### Legend
- ✅ **PHASE 1** - Updated with Phase 1 discoveries
- ⚠️ **98% DONE** - Has remaining sync issues
- ✅ **CURRENT** - Up to date with system state
- ✅ **ALIGNED** - Original documentation still accurate
- 📝 **NEEDS UPDATE** - Requires Phase 1 information

## Core Glossary (Phase 1 Additions)

### Architectural Terms

**Scene Graph Connection**: The critical discovery that ViewportState.rootEntity must contain SceneManager.rootEntity as a child for entities to be visible. Without this connection, entities exist in memory but don't render.

**Entity.realityEntity Bridge**: The direct connection between your simplified Entity wrapper and RealityKit's Entity. Phase 1 proved this must be used directly, not duplicated through factories.

**PlatformColor**: Type alias for cross-platform color compatibility (`NSColor` on macOS, `UIColor` on iOS). Essential for colored primitives.

**Task Deferral Pattern**: Using `Task { @MainActor in ... }` to avoid "Publishing changes from within view updates" errors on iOS.

**Direct Property Observation**: Observing @Published properties without throttling for real-time synchronization. Reduced grid lag from 16ms to <1ms.

### Phase 1 Discoveries

**ViewportEntityFactory Problem**: Was creating duplicate empty entities instead of using the configured entity.realityEntity, causing invisible primitives.

**Two-Graph Problem**: SceneManager.rootEntity and ViewportState.rootEntity existed as separate, unconnected scene graphs.

**Publishing Changes Error**: SwiftUI update timing conflicts on iOS required deferred creation pattern.

## Quick Reference Matrix v7.0

| Information Needed | Primary Module | Secondary Module | Status |
|-------------------|----------------|------------------|--------|
| **Phase 1 Status** | Implementation.md v5.0 | MetalRendering.md v2.0 | 98% Complete |
| **Primitive Creation** | EntitySystem.md v3.0 | Entity.swift | ✅ Working |
| **Scene Graph Fix** | ViewportState.md v5.0 | ViewportView.swift | ✅ Connected |
| **Grid Sync Issues** | MetalRendering.md v2.0 | ViewportMetalGrid.swift | ⚠️ 2% Remaining |
| **Platform Fixes** | Implementation.md v5.0 | Multiple files | ✅ Complete |
| **Debug Helpers** | ViewportState.md v5.0 | Phase 1 additions | ✅ Added |

## Standard Code Patterns (Phase 1 Validated)

### Scene Graph Connection Pattern
```swift
// CRITICAL: Connect scene graphs for visibility
public func setSceneManager(_ manager: SceneManager) {
    self.sceneManager = manager
    rootEntity.addChild(manager.rootEntity)  // THE KEY
}

// In ViewportView setup
viewportState.setSceneManager(sceneManager)  // ✅ Makes entities visible
```

### Colored Primitive Pattern
```swift
// Platform-agnostic colors
#if os(macOS)
typealias PlatformColor = NSColor
#else
typealias PlatformColor = UIColor
#endif

// Create visible primitives
static func box(size: Float = 1.0, color: PlatformColor = .systemBlue) -> Entity {
    let mesh = MeshResource.generateBox(size: size)
    var material = SimpleMaterial()
    material.color = .init(tint: color)
    return model(mesh: mesh, materials: [material])
}
```

### Task Deferral Pattern (iOS)
```swift
// Avoid publishing changes errors
private func addPrimitive(_ type: PrimitiveType) {
    Task { @MainActor in  // Defer to next run loop
        let primitive = Entity.box(...)
        sceneManager.addEntity(primitive)
        viewportState.needsUpdate = true
    }
}
```

### Direct Entity Usage Pattern
```swift
// DON'T create duplicates
// ✗ ViewportEntityFactory.createRealityEntity(for: entity)

// DO use configured entity directly
public func addEntity(_ entity: any SceneEntity) {
    let realityEntity = entity.realityEntity  // ✅ Use existing
    rootEntity.addChild(realityEntity)
}
```

## Performance Baseline (Phase 1 Updated)

```yaml
Frame Budget: 16.67ms (60fps)
Current Performance: 11-15ms
Headroom: 1-5ms

Breakdown:
  Sky Rendering: 0.5ms (Metal GPU)
  Grid Rendering: 0.3ms (Metal GPU)
  Grid Sync Overhead: 0.5-1ms (ISSUE) ← 2% remaining
  Entity Updates: 1-2ms (CPU)
  RealityKit Scene: 8-12ms (Mixed)
  SwiftUI: 2ms (CPU)
  Floating UI: ~0.5ms (negligible)

iOS: Timer at 60fps (working)
macOS: On-demand updates (working)

Status: ✅ 60fps achieved, ⚠️ Grid sync not frame-perfect
```

## Project Status (Phase 1)

### Phase 1 Achievement (98% Complete)
```yaml
Goal: Fix invisible entities issue
Status: 98% COMPLETE

Completed (98%):
  ✅ Primitives appear immediately
  ✅ Distinct colors for each type
  ✅ Scene graph properly connected
  ✅ iOS publishing errors fixed
  ✅ Platform compatibility achieved
  ✅ 60fps performance maintained
  
Remaining (2%):
  ⚠️ Frame-perfect grid synchronization
  - Minor lag during rapid camera movement
  - Occasional desync on iOS
  - Professional polish needed
```

### Core Systems Status

| System | Phase 1 Status | What Changed | Remaining |
|--------|---------------|--------------|-----------|
| **Entity Wrapper** | ✅ Complete | Added primitive factories | None |
| **Scene Graph** | ✅ Fixed | Connected via setSceneManager() | None |
| **Metal Grid** | ⚠️ 98% | Reduced lag from 16ms to <1ms | Frame-perfect sync |
| **SceneManager** | ✅ Fixed | Uses entity.realityEntity directly | None |
| **ViewportState** | ✅ Enhanced | Added scene connection | None |
| **iOS Updates** | ✅ Fixed | Timer-based with Task deferral | None |

## Known Issues (Phase 1 Status)

### 🔴 Critical for Phase 1 Completion (2%)
| ID | Issue | Impact | Status |
|----|-------|--------|--------|
| SYNC-001 | Grid lag during rapid movement | High | <1ms lag remaining |
| SYNC-002 | iOS timer desync | Medium | Functional but not optimal |
| SYNC-003 | Platform timing differences | Low | Works but inconsistent |

### ✅ Fixed in Phase 1
| ID | Issue | Resolution | Status |
|----|-------|------------|--------|
| P1-001 | Entities invisible | Scene graph connection | ✅ Fixed |
| P1-002 | Publishing changes error | Task deferral | ✅ Fixed |
| P1-003 | Factory duplication | Direct usage | ✅ Fixed |
| P1-004 | Platform colors | PlatformColor alias | ✅ Fixed |
| P1-005 | Grid major lag | Direct observation | ✅ Improved |

### 🟡 Minor Issues (Post-Phase 1)
| ID | Issue | Impact | Priority |
|----|-------|--------|----------|
| GIZ-001 | Gizmo smoothing | Low | Polish |
| OUT-001 | Outliner organization | Low | UX |

## Phase 1 Lessons Learned

### Critical Discoveries
1. **Scene graphs must be connected** - Two separate graphs don't render
2. **Don't duplicate entities** - Use entity.realityEntity directly
3. **Platform differences matter** - Always use conditional compilation
4. **Publishing timing is tricky** - Defer with Task on iOS
5. **Direct observation beats throttling** - For real-time sync

### Architectural Validations
- ✅ Simplified Entity wrapper pattern works
- ✅ Bridge to RealityKit.Entity successful
- ✅ Platform-specific update strategies necessary
- ✅ YAGNI philosophy proven correct

## Next Steps

### Complete Phase 1 (2% Remaining)
```yaml
Required for Completion:
  - Achieve frame-perfect grid/camera sync
  - Eliminate all visible lag
  - Test on all platforms
  - Update completion status to 100%

Potential Solutions:
  - Unified update loop
  - CADisplayLink synchronization
  - Direct matrix updates without Combine
```

### Phase 2: Lighting System
```yaml
Prerequisites: Phase 1 100% complete
Goals:
  - Three-point lighting
  - Shadow rendering
  - Light gizmos
Timeline: After grid sync fixed
```

### Phase 3: Material Editor
```yaml
Prerequisites: Phase 2 complete
Goals:
  - Material property UI
  - Texture loading
  - Live preview
Timeline: 2-3 weeks after Phase 2
```

## Documentation Philosophy

### Phase 1 Documentation Standards
```yaml
Good Phase 1 Documentation:
  ✅ Shows what was broken and how it was fixed
  ✅ Documents the discovery process
  ✅ Admits remaining issues (2% grid sync)
  ✅ Provides debug helpers
  ✅ Explains architectural discoveries

Bad Documentation:
  ❌ Claims 100% when it's 98%
  ❌ Hides sync issues
  ❌ Forgets the journey
  ❌ Loses critical discoveries
```

---

## 📊 Phase 1 Summary

**Phase 1 is 98% complete.** The critical invisible entity issue has been completely resolved through:

- ✅ **Scene graph connection** via setSceneManager()
- ✅ **Colored primitive factories** with platform compatibility
- ✅ **Direct entity usage** without duplication
- ✅ **iOS publishing fix** with Task deferral
- ⚠️ **Grid synchronization** improved but not perfect (2% remaining)

**Critical Remaining Work:** Achieve frame-perfect grid/camera synchronization to complete Phase 1.

**Philosophy Validated:** The simplified wrapper approach with YAGNI principles has proven successful. The system can grow from this solid foundation.

**Result:** A functional 3D editor with immediate primitive visibility, ready for final synchronization polish.