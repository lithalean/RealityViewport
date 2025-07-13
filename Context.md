# RealityViewport — AI Context Document

**Status**: Active Development - Core Features Working
**Purpose**: Accurate context for AI systems working with RealityViewport codebase
**Last Updated**: July 2025

---

## 🎯 Project Identity

**RealityViewport** is a SwiftUI + RealityKit 3D scene editor for the Orchard game engine.

**Key Facts:**

* Part of Orchard (not "RealityEditor" or standalone)
* Cross-platform: macOS, iOS, and tvOS support
* Pure SwiftUI (no AppKit/UIKit dependencies)
* Uses latest RealityKit for rendering
* Apple Silicon optimized but Intel compatible

---

## 🏗️ Current Architecture

### Directory Structure

```
RealityViewport/
├── App/
│   ├── ContentView.swift
│   └── RealityViewportApp.swift
├── Assets.xcassets/
├── Core/
│   ├── Components/
│   ├── Data/
│   ├── Extensions/
│   ├── Managers/
│   ├── Nodes/
│   ├── Systems/
│   └── Utilities/
├── Inspector/
├── Sources/Shared/
│   ├── Extensions/
│   └── PlatformColor.swift
├── Viewport/
├── Views/
│   ├── Console/
│   ├── iPhone/
│   ├── Mac/
│   └── ProjectBrowserView.swift
└── info.plist
```

---

## 🔧 Core Implementation Details

### File System Integration

RealityViewport is in the process of integrating save/load/export via a modular `ProjectManager`.

#### ✅ Implemented

* `ProjectManager` exists with full async API: `createNewProject`, `openProject`, `saveProject`, `loadProject`, `exportScene`
* `RealityViewportApp.swift` creates `SceneManager` and `ProjectManager` and injects them via `.environmentObject`
* `ContentView.swift` toggles `ProjectBrowserView` sheet using a "Projects" button
* `SceneManager.importModel(from:)` is used throughout app for asset integration

#### ⚠️ In Progress

* `ViewportToolbar.swift` does **not yet** access or use `ProjectManager` — save/load buttons are missing or unhooked
* No `.rvproject` files or directories are yet created on macOS (`~/Documents/RealityViewport Projects`) or iOS sandbox
* Export logic is implemented in `ProjectManager` but actual UI/UX for export format selection is not fully wired

#### ❌ Not Yet Started

* Auto-save system
* Resource tracking for imported USDZ files
* Project thumbnails
* Templates system logic

#### iOS-Specific Notes

Info.plist additions have not been confirmed:

```xml
<key>UIFileSharingEnabled</key><true/>
<key>LSSupportsOpeningDocumentsInPlace</key><true/>
<key>UISupportsDocumentBrowser</key><true/>
```

#### Swift Usage Example

```swift
Task {
  try await projectManager.saveProject(sceneManager)
  try await projectManager.loadProject(into: sceneManager)
}
```

> ℹ️ Note: Scene saves structure only — not binary model data. USDZ must be manually copied to the project folder.

---

## 🚧 Current Development State

### ✅ What's Working

* Platform routing (macOS/iOS/tvOS)
* Scene with multiple cameras and lights
* USDZ/Reality file import
* Dual interaction modes with toolbar control
* Transform gizmos (visual display)
* Billboard UI system
* Camera orbit/pan controls
* Scene hierarchy display
* Add node menu (cameras, lights, future primitives)
* Zoom on iOS (pinch gesture)
* Zoom on macOS (scroll wheel + pinch gesture)
* Mode switching via toolbar
* SceneManager integration with asset import

### 🔄 In Progress

* Gizmo hit detection and dragging
* Primitive mesh generation for ModelNode
* Multi-selection gestures
* Scene persistence with SwiftData
* ViewportToolbar → ProjectManager hook-up
* Export UI (format selector)
* Project thumbnails and templates

---

## 📝 Recent Updates (July 2025)

* Implemented full ProjectManager API (create, load, save, export)
* Added ProjectManager + SceneManager to RealityViewportApp.swift
* ContentView now toggles ProjectBrowserView for project interaction
* SceneManager.importModel used in multiple views
* Export destination and format system defined in ProjectManager (UI not yet complete)

---

*This document reflects actual implementation as of July 2025*
