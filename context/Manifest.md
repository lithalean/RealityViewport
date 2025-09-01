# RealityViewport Context Engineering Manifest

**Module**: MANIFEST.md  
**Version**: 7.1  
**Philosophy**: Apple-native Unity DOTS alternative  
**Architecture**: Simplified Entity Wrapper + Metal Rendering + Floating UI  
**Status**: Phase 1 - 98% Complete (Grid Sync Blocker)  
**Last Updated**: September 1 2025

## 🎯 Project Philosophy

**"What if Apple made Unity with DOTS?"**

RealityViewport is an **Apple-native 3D editor** that combines:
- **Simplified Entity wrappers** that grow with your needs
- **RealityKit's powerful ECS** as the foundation
- **Metal GPU rendering** for atmospheric effects
- **Beautiful floating glass UI** maximizing viewport space
- **Immediate primitive visibility** with colored materials ✅

This is not about building everything on day one. It's about shipping a working alpha with a solid foundation that can evolve into whatever your game needs.

## 🚀 Phase 1 Status: 98% Complete (BLOCKED)

### What's Complete (98%)
- ✅ **Primitive Visibility Fixed** - Colored primitives appear immediately
- ✅ **Scene Graph Connected** - ViewportState.setSceneManager() implemented
- ✅ **Platform Compatibility** - iOS/macOS working with proper color types
- ✅ **Publishing Errors Fixed** - Task deferral pattern working
- ✅ **60fps Performance** - Maintained with headroom

### Critical Blocker (2%)
- ❌ **Grid Synchronization on Startup** - Metal grid completely desynchronized
  - **Issue**: Grid and entities move independently on initial launch
  - **Workaround**: Close window (red button) and reopen fixes it perfectly
  - **Impact**: Professional polish missing, poor first impression
  - **Attempts**: 5+ different approaches tried, none successful
  - **Status**: Root cause unknown, appears to be initialization sequence issue

## ⚠️ CRITICAL KNOWN ISSUE

```yaml
Metal Grid Desynchronization on Startup:
  
Symptoms:
  - On first launch, Metal grid doesn't sync with camera movements
  - Entities and grid separate when rotating view
  - Makes editor feel broken initially
  
Workaround (macOS only):
  1. Close window with red button (don't quit app)
  2. Reopen window from dock
  3. Perfect sync achieved and maintained
  
iOS Status:
  - No workaround available (can't close without quitting)
  - Issue persists throughout session
  
Debugging Status:
  - Extensive debugging completed (December 2024)
  - Multiple approaches attempted and failed
  - Issue is in initialization, not update logic
  - Close/reopen triggers unknown fix we can't replicate
```

## 📚 Document Reading Guide

### For Different Purposes

#### 🎯 "I want to understand the system"
1. **Architecture.md** - Design philosophy and patterns
2. **EntitySystem.md v3.0** - Simplified Entity wrapper with primitives
3. **ViewportState.md v5.0** - Scene graph connection discovery
4. **Implementation.md v5.1** - Current status with sync blocker

#### 🐛 "I need to fix the grid sync bug"
1. **MetalRendering.md v2.1** - COMPLETE debugging documentation
2. **Implementation.md v5.1** - List of attempted solutions
3. Look for what close/reopen does that we're missing

#### 🚀 "I want to add a feature"
1. Consider if Phase 1 should complete first
2. **Architecture.md** - Patterns to follow (especially YAGNI)
3. **EntitySystem.md v3.0** - How primitives were added
4. **Visual.md** - UI integration patterns

## Module Version Registry

| Module | Version | Status | Last Updated | Key Changes |
|--------|---------|--------|--------------|-------------|
| **IMPLEMENTATION.md** | 5.1 | ⚠️ BLOCKED | December 2024 | Grid sync blocker documented |
| **METAL_RENDERING.md** | 2.1 | ⚠️ BLOCKED | December 2024 | Sync debugging complete |
| **ENTITY_SYSTEM.md** | 3.0 | ✅ PHASE 1 | December 2024 | Colored primitive factories |
| **VIEWPORTSTATE.md** | 5.0 | ✅ PHASE 1 | December 2024 | setSceneManager() connection |
| **ARCHITECTURE.md** | 3.0 | ✅ CURRENT | August 2025 | Apple-native Unity DOTS philosophy |
| **NAVIGATION.md** | 5.0 | ✅ CURRENT | August 2025 | Floating UI flows |
| **VISUAL.md** | 4.0 | ✅ CURRENT | August 2025 | Glass morphism design |
| **GESTURES.md** | 3.1 | ✅ CURRENT | August 2025 | iOS fixes documented |
| **EVOLUTION.md** | 4.0 | 📝 NEEDS UPDATE | August 2025 | Missing Phase 1 changes |
| **INTEGRATION.yaml** | 3.0 | 📝 NEEDS UPDATE | August 2025 | Missing Phase 1 integrations |
| **FILEOPERATIONS.md** | 2.0 | ✅ ALIGNED | August 2025 | Entity serialization |

### Legend
- ⚠️ **BLOCKED** - Has critical issue preventing completion
- ✅ **PHASE 1** - Updated with Phase 1 discoveries
- ✅ **CURRENT** - Up to date with system state
- 📝 **NEEDS UPDATE** - Requires Phase 1 information

## Core Glossary (Phase 1 Additions)

### Critical Terms

**Grid Sync Issue**: The critical blocker preventing Phase 1 completion. Metal grid is completely desynchronized with camera on initial launch, only fixed by closing and reopening the window.

**Close/Reopen Workaround**: The only known fix for grid sync. Closing the window (red button, not quitting) and reopening establishes proper Metal/RealityKit synchronization through an unknown mechanism.

**Scene Graph Connection**: The critical discovery that ViewportState.rootEntity must contain SceneManager.rootEntity as a child for entities to be visible.

**Entity.realityEntity Bridge**: The direct connection between your simplified Entity wrapper and RealityKit's Entity that must be used directly, not duplicated.

## Quick Reference Matrix v7.1

| Information Needed | Primary Module | Secondary Module | Status |
|-------------------|----------------|------------------|--------|
| **Grid Sync Issue** | MetalRendering.md v2.1 | Implementation.md v5.1 | ⚠️ BLOCKED |
| **Phase 1 Status** | Implementation.md v5.1 | This Manifest | 98% Complete |
| **Debugging Attempts** | MetalRendering.md v2.1 | - | Documented |
| **Primitive Creation** | EntitySystem.md v3.0 | Entity.swift | ✅ Working |
| **Scene Graph Fix** | ViewportState.md v5.0 | ViewportView.swift | ✅ Connected |

## Performance Baseline (Phase 1 Updated)

```yaml
Frame Budget: 16.67ms (60fps)
Current Performance: 11-15ms (after close/reopen)
Headroom: 1-5ms

Breakdown:
  Sky Rendering: 0.5ms (Metal GPU)
  Grid Rendering: 0.3ms (Metal GPU)
  Grid Sync Issue: N/A (not performance, initialization)
  Entity Updates: 1-2ms (CPU)
  RealityKit Scene: 8-12ms (Mixed)
  SwiftUI: 2ms (CPU)
  Floating UI: ~0.5ms (negligible)

Status: 
  ✅ 60fps achieved (after workaround)
  ❌ Grid sync broken on startup
```

## Project Status (Phase 1)

### Phase 1 Achievement (98% Complete - BLOCKED)
```yaml
Goal: Fix invisible entities issue
Status: 98% COMPLETE - BLOCKED BY GRID SYNC

Completed (98%):
  ✅ Primitives appear immediately
  ✅ Distinct colors for each type
  ✅ Scene graph properly connected
  ✅ iOS publishing errors fixed
  ✅ Platform compatibility achieved
  ✅ 60fps performance maintained
  
Critical Blocker (2%):
  ❌ Metal grid desynchronized on startup
  - Close/reopen workaround required
  - Root cause unknown after extensive debugging
  - Blocks Phase 1 completion
```

## Known Issues (Phase 1 Status)

### 🔴 Critical Blocker
| ID | Issue | Impact | Status |
|----|-------|--------|--------|
| SYNC-001 | Grid completely desynchronized on startup | CRITICAL | No fix found |

### ✅ Fixed in Phase 1
| ID | Issue | Resolution | Status |
|----|-------|------------|--------|
| P1-001 | Entities invisible | Scene graph connection | ✅ Fixed |
| P1-002 | Publishing changes error | Task deferral | ✅ Fixed |
| P1-003 | Factory duplication | Direct usage | ✅ Fixed |
| P1-004 | Platform colors | PlatformColor alias | ✅ Fixed |

## Next Steps

### Option 1: Continue Debugging Grid Sync
```yaml
Approach:
  - Investigate window server APIs
  - Research CAMetalLayer initialization
  - Try alternative Metal integration
Time: Unknown (could be days/weeks)
Risk: May never find solution
```

### Option 2: Document and Proceed
```yaml
Approach:
  - Accept close/reopen as temporary workaround
  - Document prominently for users
  - Move to Phase 2 with known issue
Time: Immediate
Risk: Poor user experience on first launch
```

### Option 3: Alternative Architecture
```yaml
Approach:
  - Use RealityKit for grid instead of Metal
  - Unified rendering pipeline
  - Major refactoring required
Time: 1-2 weeks
Risk: May lose performance advantages
```

## Phase 1 Lessons Learned

### Critical Discoveries
1. **Scene graphs must be connected** - Two separate graphs don't render
2. **Don't duplicate entities** - Use entity.realityEntity directly
3. **Platform differences matter** - Always use conditional compilation
4. **Publishing timing is tricky** - Defer with Task on iOS
5. **Metal initialization is fragile** - Close/reopen fixes unknown issue

### Debugging Insights (December 2024)
1. **Update logic is correct** - Works perfectly after close/reopen
2. **Initialization is broken** - Something about startup sequence
3. **Minimize doesn't help** - Different from close/reopen
4. **Multiple approaches failed** - Not a simple fix
5. **iOS has no workaround** - Critical platform issue

## Documentation Philosophy

### Honest Documentation
```yaml
Good Documentation:
  ✅ Admits when we don't know something
  ✅ Documents failed attempts
  ✅ Provides workarounds when available
  ✅ Clear about blockers
  ✅ Honest about completion status

Bad Documentation:
  ❌ Claims 100% when blocked
  ❌ Hides critical issues
  ❌ Pretends everything works
  ❌ Loses debugging history
```

---

## 📊 Phase 1 Summary

**Phase 1 is 98% complete but BLOCKED by a critical grid synchronization issue.**

The good:
- ✅ **Colored primitives** work beautifully
- ✅ **Scene graph** properly connected
- ✅ **Performance** excellent (after workaround)
- ✅ **Cross-platform** support achieved

The blocker:
- ❌ **Metal grid** completely desynchronized on startup
- ❌ **Only workaround** is close/reopen (not available on iOS)
- ❌ **Root cause** unknown despite extensive debugging

**Critical Decision Required:** Continue debugging, document workaround and proceed, or consider architectural change.

**Result:** A functional 3D editor that requires a workaround on first launch.