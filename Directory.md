lithalean@Tylers-Mac-mini ~ % cd /Users/lithalean/Documents/Developer/3_Build/Xcode/RealityViewport
lithalean@Tylers-Mac-mini RealityViewport % tree
.
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
│   │   │   ├── LightData.swift
│   │   │   └── TransformData.swift
│   │   ├── Extensions
│   │   │   ├── HelperExtensions.swift
│   │   │   ├── Math+Matrix.swift
│   │   │   ├── MathExtensions.swift
│   │   │   └── NotificationExtension.swift
│   │   ├── Managers
│   │   │   ├── ControlManager.swift
│   │   │   ├── ProjectManager.swift
│   │   │   ├── SceneManager.swift
│   │   │   └── SelectionManager.swift
│   │   ├── Nodes
│   │   │   ├── CameraNode.swift
│   │   │   ├── LightNode.swift
│   │   │   ├── ModelNode.swift
│   │   │   └── SceneNode.swift
│   │   ├── Systems
│   │   │   └── BillboardSystem.swift
│   │   └── Utilities
│   │       ├── GridHelper.swift
│   │       └── SceneStatistics.swift
│   ├── en.lproj
│   ├── info.plist
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
│   │       └── PlatformColor.swift
│   ├── Viewport
│   │   ├── CameraController.swift
│   │   ├── ViewportAxisHelper.swift
│   │   ├── ViewportControlSystem.swift
│   │   ├── ViewportEntityFactory.swift
│   │   ├── ViewportGrid.swift
│   │   ├── ViewportIconFactory.swift
│   │   ├── ViewportState.swift
│   │   ├── ViewportToolbar.swift
│   │   ├── ViewportView.swift
│   │   └── ViewportView+Controls.swift
│   └── Views
│       ├── Console
│       │   └── FileSystemModel.swift
│       ├── FileSystem.swift
│       ├── iPhone
│       │   └── iPhoneView.swift
│       ├── Mac
│       │   └── MacView.swift
│       └── ProjectBrowserView.swift
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

35 directories, 58 files
lithalean@Tylers-Mac-mini RealityViewport % 