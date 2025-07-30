# RealityViewport Context Engineering Manifest

**Purpose**: Master reference for AI systems to understand the context module structure  
**Last Updated**: July 2025  
**Manifest Version**: 2.0

## Module Version Registry

| Module | Version | Last Modified | Size | Health |
|--------|---------|--------------|------|---------|
| ARCHITECTURE.md | 2.0 | July 2025 | 7KB | ✅ |
| IMPLEMENTATION.md | 2.0 | July 2025 | 9KB | ✅ |
| VISUAL.md | 2.0 | July 2025 | 5KB | ✅ |
| NAVIGATION.md | 2.0 | July 2025 | 6KB | ✅ |
| INTEGRATION.yaml | 1.0 | July 2025 | 2KB | ✅ |
| EVOLUTION.md | 2.0 | July 2025 | 5KB | ✅ |
| VIEWPORTSTATE.md | 1.0 | July 2025 | 4KB | ✅ |
| FILEOPERATIONS.md | 1.0 | July 2025 | 5KB | ✅ |
| GESTURES.md | 1.0 | July 2025 | 4KB | ✅ |
| session.json | - | DYNAMIC | 2KB | ✅ |

## Quick Reference Matrix

| Information Needed | Primary Module | Secondary Module |
|-------------------|----------------|------------------|
| How something works | ARCHITECTURE.md | IMPLEMENTATION.md |
| Current state/status | IMPLEMENTATION.md | session.json |
| Visual appearance | VISUAL.md | VIEWPORTSTATE.md |
| User flow/navigation | NAVIGATION.md | FILEOPERATIONS.md |
| External connections | INTEGRATION.yaml | ARCHITECTURE.md |
| Past decisions/why | EVOLUTION.md | ARCHITECTURE.md |
| Recent changes | session.json | EVOLUTION.md |
| Viewport rendering | VIEWPORTSTATE.md | ARCHITECTURE.md |
| File operations | FILEOPERATIONS.md | NAVIGATION.md |
| Input handling | GESTURES.md | NAVIGATION.md |

## Context Loading Strategy

### For Bug Fixes
1. IMPLEMENTATION.md (current state)
2. session.json (recent changes)
3. ARCHITECTURE.md (design constraints)
4. VIEWPORTSTATE.md (if viewport-related)

### For New Features
1. ARCHITECTURE.md (design patterns)
2. NAVIGATION.md (user flow)
3. VISUAL.md (UI requirements)
4. IMPLEMENTATION.md (integration points)
5. FILEOPERATIONS.md (if file-related)

### For Refactoring
1. EVOLUTION.md (past decisions)
2. ARCHITECTURE.md (patterns)
3. IMPLEMENTATION.md (current state)
4. VIEWPORTSTATE.md (state management)

## Project-Specific Context

### RealityViewport Key Facts
- Part of Orchard game engine ecosystem
- SwiftUI + RealityKit 3D scene editor
- Cross-platform: macOS, iOS, tvOS
- Pure SwiftUI (no AppKit/UIKit dependencies)
- Apple Silicon optimized
- ~70% implementation complete with mature architecture

### Critical Components
- **ViewportState**: Central 3D viewport state management
- **FileDialogs**: Full file system integration (85% complete)
- **BaseSceneNode**: Sophisticated node hierarchy system
- **Dual Selection**: Entity/Node parallel tracking
- **Transform Gizmos**: Visual manipulation tools