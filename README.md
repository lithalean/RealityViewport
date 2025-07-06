# 🚀 RealityViewport

*Next-Generation 3D Scene Editor for Apple Silicon*

![Platform](https://img.shields.io/badge/platform-macOS%2026%2B-blue)
![Architecture](https://img.shields.io/badge/arch-ARM64%20ONLY-green)
![Swift Version](https://img.shields.io/badge/swift-6.0%2B-orange)
![Xcode](https://img.shields.io/badge/Xcode-26%2B-red)

**RealityViewport** is the scene editor component of Orchard—a cutting-edge 3D editor built exclusively for Apple Silicon and the latest Apple technologies.

---

## ✨ Features

- **Modern RealityKit Pipeline**: Latest rendering with persistent entities
- **Advanced Selection System**: Multi-selection with visual feedback  
- **Transform Gizmos**: Translate, rotate, scale with precise control
- **Cross-Platform**: Native macOS and iOS experiences
- **Dual Interaction Modes**: Environment (camera) vs Entity (object) control
- **Real-time Viewport**: Live scene updates with 120fps targets

## 🎯 Built for the Future

This project uses APIs that **only exist** in the 2025+ Apple ecosystem:
- RealityKit 5 rendering pipeline
- Swift 6 concurrency features
- Modern SwiftUI 5 layout system
- Apple Silicon-exclusive Metal 3 features
- Advanced file coordination APIs

> ⚡ **Requirements**  
> - **macOS 26+** or **iOS 26+**
> - **Apple Silicon** (M1/M2/M3/M4 only)
> - **Xcode 26+**
> - **Swift 6**

> ❌ **Not Supported**  
> - Intel Macs (will not compile)
> - Legacy macOS/iOS versions
> - Rosetta 2 compatibility

---

## 🏗️ Architecture

```
RealityViewport/
├── Core/                   # Swift 6 business logic
│   ├── Nodes/             # Actor-based scene graph
│   ├── Managers/          # @Observable state management  
│   └── Components/        # Transform gizmos & tools
├── Viewport/              # RealityKit rendering engine
├── Inspector/             # SwiftUI property system
└── Views/                 # Platform-adaptive layouts
    ├── Mac/               # macOS split-view interface
    └── iPhone/            # iOS overlay interface
```

**Key Components:**
- **Scene Graph**: Camera, Light, and Model nodes with full transform control
- **Camera System**: Orbit, pan, fly modes with gamepad support
- **Selection Manager**: Multi-object selection with gizmo transforms
- **Platform UI**: Adaptive interface for Mac (split view) and iPhone (overlays)

---

## 🚧 Development Status

### ✅ Phase 1: Foundation (Complete)
- RealityKit viewport with persistent entities
- Selection system with visual feedback  
- Transform gizmos (translate, rotate, scale)
- Cross-platform architecture (macOS/iOS)

### 🚀 Phase 2: Scene Management (In Progress)
- Multi-selection with modern gestures
- Scene graph persistence & undo/redo
- Advanced node relationships
- iPhone adaptive inspector panels
- Unified icon-only toolbars

### 🔮 Phase 3: Advanced Features (Planned)
- Reality Composer Pro integration
- Advanced lighting & materials
- Asset import pipeline
- Scene templates & presets

---

## 💻 Getting Started

### Prerequisites
- Mac with Apple Silicon (M1 minimum)
- macOS 26 Developer Beta or later
- Xcode 26 Developer Beta or later  
- Active Apple Developer account

### Quick Start
1. Clone the repository
2. Open `RealityViewport.xcodeproj` in Xcode 26+
3. Select your target device (Mac or iPhone)
4. Build and run

### Basic Usage
- **Environment Mode**: Camera navigation (orbit, pan, zoom)
- **Entity Mode**: Object selection and transformation
- **Import**: Drag USDZ or Reality files into the viewport
- **Transform**: Use gizmos to move, rotate, scale objects
- **Inspect**: Use the side panel to view/edit properties

---

## 🎮 Controls

### macOS
- **Mouse**: Orbit camera (drag), zoom (scroll)
- **Keyboard**: Mode switching, selection cycling  
- **Shortcuts**: Standard Mac conventions

### iOS  
- **Touch**: Tap to select, drag to transform
- **Gestures**: Pinch to zoom, pan to orbit
- **Interface**: Sliding inspector panel

---

## 🔧 Part of Orchard

RealityViewport is the scene editor component of **Orchard**—a complete Apple-native game development ecosystem built for modern Apple devices.

**Related Components:**
- Asset pipeline & import tools
- Reality Composer Pro integration  
- Metal 3 rendering optimizations
- SwiftUI-based game logic editor

---

## 📋 Requirements

| Component | Version |
|-----------|---------|
| macOS | 26+ |
| iOS | 26+ |  
| Xcode | 26+ |
| Swift | 6.0+ |
| Hardware | Apple Silicon only |

---

## 🤝 Contributing

This is a cutting-edge project using the latest Apple technologies. Contributions should:
- Use Swift 6 concurrency features
- Follow Apple's latest design patterns
- Maintain Apple Silicon optimization
- Support both macOS and iOS platforms

---

## 📄 License

[Add your license information here]

---

*Built with ❤️ for the Apple ecosystem*