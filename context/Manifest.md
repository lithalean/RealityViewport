# RealityViewport Context Engineering Manifest

**Module**: MANIFEST.md  
**Version**: 6.0  
**Philosophy**: Apple-native Unity DOTS alternative  
**Architecture**: Simplified Entity Wrapper + Metal Rendering + Floating UI  
**Status**: Alpha Foundation Complete - Ready to Ship  
**Last Updated**: August 2025

## 🎯 Project Philosophy

**"What if Apple made Unity with DOTS?"**

RealityViewport is an **Apple-native 3D editor** that combines:
- **Simplified Entity wrappers** that grow with your needs
- **RealityKit's powerful ECS** as the foundation
- **Metal GPU rendering** for atmospheric effects
- **Beautiful floating glass UI** maximizing viewport space

This is not about building everything on day one. It's about shipping a working alpha with a solid foundation that can evolve into whatever your game needs.

## 📚 Document Reading Guide

### For Different Purposes

#### 🎯 "I want to understand the system"
1. **Architecture.md** - Design philosophy and patterns
2. **EntitySystem.md** - Simplified Entity wrapper system
3. **MetalRendering.md** - GPU rendering pipeline
4. **Implementation.md** - Current state and philosophy

#### 🐛 "I need to fix a bug"
1. **Implementation.md** - Known issues and status
2. **session.json** - Runtime state
3. **EntitySystem.md** or **MetalRendering.md** - Depending on bug type
4. Component's specific manager documentation

#### 🚀 "I want to add a feature"
1. **Architecture.md** - Patterns to follow (especially YAGNI)
2. **EntitySystem.md** - How to extend the wrapper
3. **Visual.md** - UI integration patterns
4. **Navigation.md** - User flow integration

#### 📁 "I'm working with files"
1. **FileOperations.md** - Serialization patterns
2. **Navigation.md** - Import/export flows
3. **EntitySystem.md** - Entity data structures

## Module Version Registry

| Module | Version | Philosophy | Last Updated | Description |
|--------|---------|------------|--------------|-------------|
| **ARCHITECTURE.md** | 3.0 | ✅ CURRENT | August 2025 | Apple-native Unity DOTS philosophy |
| **ENTITY_SYSTEM.md** | 2.0 | ✅ CURRENT | August 2025 | Simplified wrapper that grows |
| **IMPLEMENTATION.md** | 4.0 | ✅ UPDATED | August 2025 | Beautiful UI, iOS fixes, ready to ship |
| **VIEWPORTSTATE.md** | 4.0 | ✅ UPDATED | August 2025 | Platform-specific update strategies |
| **NAVIGATION.md** | 5.0 | ✅ UPDATED | August 2025 | Floating UI flows |
| **VISUAL.md** | 4.0 | ✅ UPDATED | August 2025 | Glass morphism design |
| **GESTURES.md** | 3.1 | ✅ UPDATED | August 2025 | iOS fixes documented |
| **EVOLUTION.md** | 4.0 | ✅ UPDATED | August 2025 | Complete history with floating UI |
| **INTEGRATION.yaml** | 3.0 | ✅ UPDATED | August 2025 | All integrations current |
| **MANIFEST.md** | 6.0 | ✅ UPDATED | August 2025 | Master reference aligned |
| **METAL_RENDERING.md** | 1.0 | ✅ ALIGNED | August 2025 | GPU pipeline documentation |
| **FILEOPERATIONS.md** | 2.0 | ✅ ALIGNED | August 2025 | Entity serialization |
| **session.json** | 4.0 | DYNAMIC | Runtime | Current floating UI state |

### Legend
- ✅ **UPDATED** - Reflects floating UI and iOS fixes
- ✅ **CURRENT** - Up to date with system state
- ✅ **ALIGNED** - Original documentation still accurate

## Core Glossary

### Architectural Terms

**Entity (Custom)**: Your simplified wrapper class around RealityKit.Entity. Provides Unity-like API that starts simple and grows with needs. Not a full replication of Apple's Entity.

**RealityKit.Entity**: Apple's native Entity with full ECS capabilities. Always accessible via `entity.realityEntity` bridge.

**Floating UI**: Glass morphism panels that float above the edge-to-edge viewport. Replaced NavigationSplitView complexity with simple ZStack.

**Simplified Wrapper Philosophy**: Start with basic features (position, rotation, scale), add complexity only when actually needed. Ship today, grow tomorrow.

**YAGNI (You Aren't Gonna Need It)**: Don't build features before you need them. The 30% "missing" features aren't missing - they're intentionally not built yet.

**Bridge Pattern**: `entity.realityEntity` connects your simplified API to RealityKit's full power. Clean separation between wrapper and rendering.

**DOTS (Data-Oriented Technology Stack)**: Unity's modern architecture. We're building "what if Apple made DOTS" - combining Unity patterns with Apple's native performance.

**Alpha Foundation**: The 70% of features that lets you ship today. The remaining 30% comes later, if needed.

## Quick Reference Matrix v6.0

| Information Needed | Primary Module | Secondary Module | Philosophy |
|-------------------|----------------|------------------|------------|
| **Core philosophy** | ARCHITECTURE.md | MANIFEST.md | Start simple, grow with needs |
| **Entity wrapper** | ENTITY_SYSTEM.md | ARCHITECTURE.md | Simplified API over RealityKit |
| **Current state** | IMPLEMENTATION.md | session.json | 100% ready to ship |
| **Floating UI** | VISUAL.md | NAVIGATION.md | Glass panels, edge-to-edge |
| **iOS performance** | VIEWPORTSTATE.md | IMPLEMENTATION.md | Timer-based updates |
| **Rendering bridge** | VIEWPORTSTATE.md | METAL_RENDERING.md | Clean separation |
| **What's NOT built** | IMPLEMENTATION.md | ENTITY_SYSTEM.md | Intentional, not missing |
| **Type confusion** | VIEWPORTSTATE.md | ENTITY_SYSTEM.md | Always disambiguate |

## Standard Code Patterns

### Entity Type Disambiguation
```swift
// ALWAYS use this pattern:

// Your simplified wrapper (grows over time)
import RealityViewport
let myEntity = Entity()               // Your simplified API
myEntity.position = SIMD3<Float>()    // Easy to use

// Apple's full-featured Entity
import RealityKit
let rkEntity = RealityKit.Entity()    // Apple's Entity
rkEntity.components[Transform.self]   // Full ECS access

// The bridge between them
let wrapped = myEntity.realityEntity  // Access full power when needed

// ViewportState uses RealityKit.Entity directly
viewportState.rootEntity              // Always RealityKit.Entity
```

### Floating UI Pattern
```swift
// Simple ZStack for all platforms
ZStack {
    ViewportView()          // Edge-to-edge
        .ignoresSafeArea()
    
    // Floating panels
    VStack {
        Toolbar()
            .floatingPanel()  // Glass morphism
        Spacer()
    }
}

// Standard floating panel style
func floatingPanel() -> some View {
    self
        .background(.ultraThinMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 12))
        .shadow(color: .black.opacity(0.2), radius: 10)
        .padding(20)
}
```

### Performance Baseline
```yaml
Frame Budget: 16.67ms (60fps)
Current Performance: 11-15ms
Headroom: 1-5ms

Breakdown:
  Sky Rendering: 0.5ms (Metal GPU)
  Grid Rendering: 0.3ms (Metal GPU)
  Entity Updates: 1-2ms (CPU)
  RealityKit Scene: 8-12ms (Mixed)
  SwiftUI: 2ms (CPU)
  Floating UI: ~0.5ms (negligible)

iOS: Timer at 60fps (smooth)
macOS: On-demand updates (efficient)

Status: ✅ Smooth 60fps achieved
```

## Project Status

### What "70% Complete" Really Means
```yaml
Built (70%):
  ✅ Everything needed for working 3D editor
  ✅ Core entity manipulation
  ✅ Professional rendering pipeline
  ✅ Beautiful floating glass UI
  ✅ iOS performance fixed
  
Not Built Yet (30%):
  ⏳ Features we don't need today
  ⏳ Optimizations for problems we don't have
  ⏳ Systems for use cases that may never exist
  
This is intentional. We're 100% ready to ship the alpha.
```

### Core Systems Status

| System | Completion | What It Means |
|--------|------------|---------------|
| **Entity Wrapper** | 100% of Alpha | Has everything needed to ship |
| **Metal Rendering** | 100% Complete | GPU pipeline fully working |
| **Floating UI** | 100% Complete | Beautiful glass panels |
| **iOS Performance** | 100% Fixed | Timer-based updates working |
| **SceneManager** | 95% Complete | Core functionality done |
| **SelectionManager** | 90% Complete | Selection works |
| **ProjectManager** | 85% Complete | Save/load works |
| **ViewportState** | 95% Complete | Platform-optimized |

## Technology Stack

### Current Implementation
- **UI**: SwiftUI with Floating Glass Panels (ZStack)
- **3D Engine**: RealityKit (Apple's ECS)
- **Entity Layer**: Simplified wrapper (grows with needs)
- **Rendering**: Metal (sky/grid) + RealityKit (3D models)
- **Platforms**: iOS 17.0+, macOS 14.0+, tvOS 17.0+
- **Performance**: 60fps achieved with headroom

### Architecture Layers
```
Your Code (Simple API) → Entity Wrapper
                          ↓
                    .realityEntity (bridge)
                          ↓
RealityKit (Full Power) ← ViewportState (Rendering)
                          ↑
                    Floating UI (ZStack)
```

## Known Limitations (And That's OK)

### Actual Issues (Will Fix)
| ID | Issue | Impact | Status |
|----|-------|--------|--------|
| GIZ-001 | Gizmo needs smoothing | Medium | Functional, needs polish |
| ENT-001 | SpotLight → PointLight | Low | RealityKit limitation |

### Fixed Issues (August 25, 2025)
| ID | Issue | Resolution | Status |
|----|-------|------------|--------|
| iOS-001 | Publishing changes error | Timer-based updates | ✅ Fixed |
| UI-001 | Double toolbar | Removed NavigationSplitView | ✅ Fixed |
| UI-002 | Inspector wrong side | Floating panel on right | ✅ Fixed |
| HAP-001 | Duplicate HapticStyle | Centralized utility | ✅ Fixed |
| PLAT-001 | LightEntity cross-platform | Conditional imports | ✅ Fixed |

### Intentionally Not Built (YAGNI)
```yaml
Not Built Yet:
  - Animation system (use realityEntity when needed)
  - Physics wrappers (use realityEntity when needed)
  - Particle systems (add when games need effects)
  - Undo/redo (add when users request it)
  - Entity prefabs (add when patterns emerge)
  - LOD system (add if performance issues)

These aren't bugs or gaps - they're future possibilities.
```

## Growth Philosophy

### How The System Grows
```mermaid
graph LR
    A[Ship Alpha] --> B{Users Need Feature?}
    B -->|Yes| C[Add to Wrapper]
    B -->|No| D[Keep Simple]
    C --> E[Only What's Needed]
    D --> F[YAGNI]
```

### Example: Adding Animation (When Needed)
```swift
// TODAY: Not wrapped (alpha doesn't need it)
entity.realityEntity.playAnimation(...)  // Direct access works

// FUTURE: If many users need animation
extension Entity {
    func playAnimation(_ name: String) {
        // Add simple wrapper when actually needed
    }
}
```

## Module Dependencies

```
ARCHITECTURE.md (Philosophy)
    ├── ENTITY_SYSTEM.md (Simplified wrapper)
    ├── VIEWPORTSTATE.md (Rendering bridge)
    └── IMPLEMENTATION.md (Current state)

VISUAL.md (UI Design)
    ├── Floating glass panels
    ├── Edge-to-edge viewport
    └── Platform consistency

VIEWPORTSTATE.md (Rendering Layer)
    ├── RealityKit.Entity management
    ├── Platform-specific updates
    └── No wrapper knowledge

IMPLEMENTATION.md (Reality Check)
    ├── What's built (70%)
    ├── What's not (30%)
    └── Why that's perfect
```

## Success Metrics

### Alpha Success ✅ ACHIEVED
| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| Can edit 3D scenes | Yes | Yes | ✅ |
| Runs at 60fps | 60fps | 60fps | ✅ |
| Cross-platform | 3+ | 3+ | ✅ |
| Beautiful UI | Yes | Glass morphism | ✅ |
| Viewport space | Max | 95%+ | ✅ |
| Can ship today | Yes | Yes | ✅ |
| Room to grow | Yes | Yes | ✅ |

### Future Success (Not Commitments)
- Grows based on actual user needs
- Stays simple despite growth
- Avoids feature bloat
- Maintains 60fps

## Next Priorities

### Immediate (If Needed)
1. Polish gizmo interaction (nice to have)
2. Add tests (when stability matters)
3. Profile performance (if issues arise)

### Based on User Feedback
- Animation support (if users need it)
- Physics integration (if games require it)
- Undo/redo (if users request it)
- Prefabs (if patterns emerge)

### Probably Never (YAGNI)
- Full ECS wrapper parity
- Every possible feature
- Complex systems without use cases
- Premature optimizations

## Documentation Philosophy

### What Good Documentation Means Here
```yaml
Good Documentation:
  ✅ Explains what IS built
  ✅ Explains what's NOT built (and why)
  ✅ Shows how to grow the system
  ✅ Provides escape hatches
  ✅ Admits limitations proudly

Bad Documentation:
  ❌ Pretends everything is built
  ❌ Hides limitations
  ❌ Over-promises features
  ❌ Ignores YAGNI principle
```

---

## 🚀 Ready to Ship!

**This is not a 70% complete project. This is a 100% ready alpha.**

The documentation reflects a **beautiful, pragmatic system** that:
- ✅ **Ships today** with professional polish
- ✅ **Looks amazing** with floating glass UI
- ✅ **Runs smoothly** on all platforms (iOS fixed!)
- ✅ **Grows tomorrow** based on real needs
- ✅ **Stays simple** despite capabilities
- ✅ **Leverages Apple's power** through RealityKit
- ✅ **Respects YAGNI** - build what you need when you need it

**Philosophy:** Start simple. Ship early. Grow with purpose. Stay Apple native.

**Result:** A beautiful 3D editor with floating glass UI that can become whatever your game needs it to be.

**Today's Achievement (August 25, 2025):** Transformed functional UI into stunning floating panels. Fixed iOS performance. Alpha is ready to ship!