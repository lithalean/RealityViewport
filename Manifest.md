# RealityViewport Context Engineering Manifest

**Purpose**: Master reference for AI systems to understand the context module structure  
**Last Updated**: July 2025  
**Manifest Version**: 1.0

## Module Version Registry

| Module | Version | Last Modified | Size | Health |
|--------|---------|--------------|------|---------|
| ARCHITECTURE.md | 1.0 | July 2025 | 4KB | ✅ |
| IMPLEMENTATION.md | 1.0 | July 2025 | 6KB | ✅ |
| VISUAL.md | 1.0 | July 2025 | 3KB | 🔄 |
| NAVIGATION.md | 1.0 | July 2025 | 3KB | 🔄 |
| INTEGRATION.yaml | 1.0 | July 2025 | 2KB | ✅ |
| EVOLUTION.md | 1.0 | July 2025 | 2KB | 🔄 |
| session.json | - | DYNAMIC | 1KB | ✅ |

## Quick Reference Matrix

| Information Needed | Primary Module | Secondary Module |
|-------------------|----------------|------------------|
| How something works | ARCHITECTURE.md | IMPLEMENTATION.md |
| Current state/status | IMPLEMENTATION.md | session.json |
| Visual appearance | VISUAL.md | ARCHITECTURE.md |
| User flow/navigation | NAVIGATION.md | IMPLEMENTATION.md |
| External connections | INTEGRATION.yaml | ARCHITECTURE.md |
| Past decisions/why | EVOLUTION.md | ARCHITECTURE.md |
| Recent changes | session.json | EVOLUTION.md |

## Context Loading Strategy

### For Bug Fixes
1. IMPLEMENTATION.md (current state)
2. session.json (recent changes)
3. ARCHITECTURE.md (design constraints)

### For New Features
1. ARCHITECTURE.md (design patterns)
2. NAVIGATION.md (user flow)
3. VISUAL.md (UI requirements)
4. IMPLEMENTATION.md (integration points)

### For Refactoring
1. EVOLUTION.md (past decisions)
2. ARCHITECTURE.md (patterns)
3. IMPLEMENTATION.md (current state)

## Project-Specific Context

### RealityViewport Key Facts
- Part of Orchard game engine ecosystem
- SwiftUI + RealityKit 3D scene editor
- Cross-platform: macOS, iOS, tvOS
- Pure SwiftUI (no AppKit/UIKit dependencies)
- Apple Silicon optimized