lithalean@Mac ~ % cd /Users/lithalean/Documents/Developer/3_Build/Xcode/RealityViewport
lithalean@Mac RealityViewport % tree
.
├── analyze_project.sh
├── RealityViewport
│   ├── App
│   │   ├── ContentView.swift
│   │   └── RealityViewportApp.swift
│   ├── Assets.xcassets
│   │   ├── AccentColor.colorset
│   │   │   └── Contents.json
│   │   ├── AppIcon.appiconset
│   │   │   └── Contents.json
│   │   └── Contents.json
│   ├── Core
│   │   ├── Components
│   │   │   ├── BillboardComponent.swift
│   │   │   ├── EditorComponents.swift
│   │   │   └── TransformGizmo.swift
│   │   ├── Data
│   │   │   ├── ExportFormat.swift
│   │   │   ├── LightData.swift
│   │   │   └── TransformData.swift
│   │   ├── Extensions
│   │   │   ├── FileDialogs.swift
│   │   │   ├── HelperExtensions.swift
│   │   │   ├── Math+Matrix.swift
│   │   │   ├── MathExtensions.swift
│   │   │   └── NotificationExtension.swift
│   │   ├── Managers
│   │   │   ├── ControlManager.swift
│   │   │   ├── DayNightManager.swift
│   │   │   ├── ProjectManager.swift
│   │   │   ├── SceneManager.swift
│   │   │   └── SelectionManager.swift
│   │   ├── MetalGridRenderer.swift
│   │   ├── MetalRenderer.swift
│   │   ├── MetalSkyRenderer.swift
│   │   ├── Shaders
│   │   │   ├── DayNightBackgroundShader.metal
│   │   │   └── GridShader.metal
│   │   ├── Systems
│   │   │   └── BillboardSystem.swift
│   │   └── Utilities
│   │       ├── GridHelper.swift
│   │       └── SceneStatistics.swift
│   ├── Entities
│   │   ├── CameraEntity.swift
│   │   ├── Entity.swift
│   │   ├── LightEntity.swift
│   │   ├── ModelEntity.swift
│   │   └── SceneEntity.swift
│   ├── Inspector
│   │   ├── CameraController+ViewportSupport.swift
│   │   ├── InspectorView.swift
│   │   ├── OutlinerView.swift
│   │   └── PropertiesView.swift
│   ├── Item.swift
│   ├── Sources
│   │   └── Shared
│   │       ├── Extensions
│   │       │   ├── FileDocumentTypes.swift
│   │       │   ├── FilePermissionsHelper.swift
│   │       │   ├── HapticFeedback.swift
│   │       │   ├── HitTesting.swift
│   │       │   ├── KeyboardShortcuts.swift
│   │       │   └── ScrollWheelHandler.swift
│   │       ├── GameColors.swift
│   │       ├── PlatformAliases.swift
│   │       ├── PlatformColor.swift
│   │       └── SharedTypes.swift
│   ├── Viewport
│   │   ├── CameraController.swift
│   │   ├── ViewportAxisHelper.swift
│   │   ├── ViewportControlSystem.swift
│   │   ├── ViewportEntityFactory.swift
│   │   ├── ViewportGrid.swift
│   │   ├── ViewportIconFactory.swift
│   │   ├── ViewportMetalGrid.swift
│   │   ├── ViewportState.swift
│   │   ├── ViewportToolbar.swift
│   │   ├── ViewportView.swift
│   │   └── ViewportView+Controls.swift
│   └── Views
│       ├── Console
│       │   └── FileSystemModel.swift
│       ├── FileSystem.swift
│       ├── MetalGridTest.swift
│       ├── MetalSkyTest.swift
│       └── ProjectBrowserView.swift
├── realityviewport_analysis_20250810_140156.txt
└── RealityViewport.xcodeproj
    ├── project.pbxproj
    ├── project.xcworkspace
    │   ├── contents.xcworkspacedata
    │   ├── xcshareddata
    │   │   └── swiftpm
    │   │       └── configuration
    │   └── xcuserdata
    │       └── lithalean.xcuserdatad
    │           └── UserInterfaceState.xcuserstate
    └── xcuserdata
        └── lithalean.xcuserdatad
            ├── xcdebugger
            │   └── Breakpoints_v2.xcbkptlist
            └── xcschemes
                └── xcschememanagement.plist

33 directories, 72 files
lithalean@Mac RealityViewport % 