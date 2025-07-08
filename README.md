# 🎬 RealityViewport

*Native 3D Scene Editor for Apple Platforms*

![Platform](https://img.shields.io/badge/platform-macOS%20%7C%20iOS%20%7C%20tvOS-blue)
![Architecture](https://img.shields.io/badge/arch-Apple%20Silicon-green)
![Swift Version](https://img.shields.io/badge/swift-6.0-orange)
![Status](https://img.shields.io/badge/status-In%20Development-yellow)

**RealityViewport** is a professional 3D scene editor built with SwiftUI and RealityKit, designed as the primary 3D editing interface for the Orchard game engine. It provides native experiences across macOS, iOS, and tvOS with a modern architecture leveraging the latest Apple technologies.

---

## ✨ Key Features

### 🎯 Modern 3D Editing
- **RealityKit-powered viewport** with 60fps+ performance
- **Dual interaction modes** - Environment (camera) vs Entity (object) control
- **Transform gizmos** - Visual manipulation for translate, rotate, scale
- **Multi-selection support** - Work with multiple objects simultaneously
- **Scene graph** - Hierarchical organization with cameras, lights, and models

### 📱 True Cross-Platform
- **macOS** - Split-view interface with resizable panels
- **iOS** - Full-screen editing with slide-out inspector
- **tvOS** - Viewport-focused experience (preview)
- Adaptive layouts that feel native on each platform

### 🚀 Advanced Architecture
- **SwiftUI + RealityKit** - No UIKit/AppKit dependencies
- **Billboard system** - UI elements that face the camera
- **Entity-node mapping** - Clean separation of concerns
- **Keyboard & gamepad** support on compatible devices

---

## 🖼️ Platform Experiences

### macOS Interface
<img width="800" alt="macOS Split View" src="https://github.com/user-attachments/assets/placeholder-macos">

- Traditional 3D editor layout (viewport left, inspector right)
- Resizable panels with drag handles
- Full keyboard shortcuts
- Hover interactions for precision

### iOS Interface
<img width="400" alt="iOS Overlay Interface" src="https://github.com/user-attachments/assets/placeholder-ios">

- Full-screen viewport with overlay controls
- Slide-in inspector from right edge
- Touch-optimized gizmo interaction
- Bottom toolbar with material background

---

## 🏗️ Technical Architecture

### Core Components
```
├── Core/
│   ├── Managers/       # SceneManager, SelectionManager
│   ├── Nodes/         # CameraNode, LightNode, ModelNode
│   ├── Components/    # TransformGizmo, BillboardComponent
│   └── Systems/       # BillboardSystem
├── Viewport/
│   ├── ViewportView.swift         # Main RealityKit view
│   ├── ViewportState.swift        # Viewport-specific state
│   ├── CameraController.swift     # Orbit/pan camera controls
│   └── ViewportEntityFactory.swift # Entity creation
└── Inspector/
    ├── OutlinerView.swift        # Scene hierarchy
    └── PropertiesView.swift      # Object properties
```

### Interaction Modes
RealityViewport features a unique dual-mode system:

```swift
enum ViewportInteractionMode {
    case environment  // Camera navigation (orbit, pan, zoom)
    case entity      // Object manipulation (select, transform)
}
```

- **Environment Mode**: Drag to orbit camera, scroll to zoom
- **Entity Mode**: Click to select, drag gizmos to transform
- Clear visual indicators show current mode

### File Support
- **USDZ** - Universal Scene Description
- **Reality** - Reality Composer files
- Drag & drop or file picker import

---

## 🚧 Development Status

### Currently Working
- ✅ Cross-platform viewport (macOS, iOS, tvOS)
- ✅ Basic scene graph with cameras, lights, models
- ✅ Dual interaction mode system
- ✅ Transform gizmos (visual feedback)
- ✅ File import (USDZ, Reality)
- ✅ Billboard UI system
- ✅ Camera orbit/pan controls

### In Progress
- 🔄 Multi-selection gestures refinement
- 🔄 Gizmo hit detection improvements
- 🔄 Scene persistence with SwiftData
- 🔄 Advanced node relationships

### Planned Features
- 📋 Undo/redo system
- 📋 Reality Composer Pro integration
- 📋 Advanced lighting controls
- 📋 Material editing
- 📋 Animation timeline

---

## 🛠️ Building from Source

*Note: Source code is currently in a private repository during initial development.*

### Requirements
- macOS 15.0+ (for development)
- Xcode 16.0+
- Swift 6.0
- Apple Silicon Mac recommended

### Supported Platforms
- macOS 15.0+
- iOS 18.0+
- tvOS 18.0+ (limited features)

---

## 🎮 Controls Reference

### macOS
- **Left Mouse**: Select objects / Orbit camera
- **Right Mouse**: Context menus
- **Scroll**: Zoom in/out
- **Space**: Toggle interaction mode
- **Delete**: Remove selected objects

### iOS
- **Tap**: Select objects
- **Drag**: Transform gizmos / Pan camera
- **Pinch**: Zoom camera
- **Two-finger drag**: Orbit camera
- **Long press**: Context menu

---

## 🔗 Part of Orchard

RealityViewport is one component of the **Orchard** game engine ecosystem:

- **RealityViewport** - 3D scene editing (this project)
- **RealitySyntax** - Code editor with syntax highlighting
- **RealityAssets** - Asset management and import
- **RealityBuild** - Build and deployment tools

Each component is designed to work standalone or integrated.

---

## 📚 Documentation

Documentation is being developed alongside the project:
- **Architecture Guide** - Understanding the codebase
- **Platform Adaptation** - macOS vs iOS differences
- **Extension Guide** - Adding new node types
- **RealityKit Integration** - Working with entities

---

## 🤝 Contributing

While the repository is currently private, we welcome feedback:
- Feature requests and suggestions
- Bug reports (once public testing begins)
- Platform-specific improvements
- Performance optimization ideas

---

## 📄 License

RealityViewport will be released under an open-source license (TBD) once it reaches public beta.

---

## 🙏 Acknowledgments

- [RealityKit](https://developer.apple.com/documentation/realitykit) team at Apple
- SwiftUI engineering for modern declarative UI
- The broader Apple developer community

---

<p align="center">
  <i>Built with ❤️ for the Apple ecosystem as part of the Orchard game engine</i>
</p>

<p align="center">
  <a href="#-realityviewport">Back to top ↑</a>
</p>