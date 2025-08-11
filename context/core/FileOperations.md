# RealityViewport File Operations Documentation

**Purpose**: Document complete file system integration with Entity/ECS architecture  
**Version**: 2.0  
**Status**: 90% Complete - Entity-Aware Operations  
**Last Updated**: August 2025

## Overview

RealityViewport implements sophisticated file system integration using SwiftUI's native file dialogs, security-scoped resources, and a custom .rvproject format. The system is fully updated for the Entity/ECS architecture with proper serialization of entities instead of nodes.

## Project File Format (v2.0)

### .rvproject Structure
```
ProjectName.rvproject
├── project.json          # Complete project data (single file approach)
└── Assets/               # Referenced model files (optional)
    ├── model1.usdz
    └── model2.reality
```

### Project Format (Entity-Based)
```json
{
  "name": "My Project",
  "url": "file:///path/to/project.rvproject",
  "dateCreated": "2025-08-10T10:00:00Z",
  "dateModified": "2025-08-10T15:30:00Z",
  "sceneData": {
    "entities": [
      {
        "id": "UUID",
        "name": "Main Camera",
        "type": "Camera",
        "transform": {
          "position": [0, 1, 5],
          "rotation": [0, 0, 0, 1],
          "scale": [1, 1, 1]
        },
        "cameraData": {
          "fov": 60.0,
          "nearPlane": 0.1,
          "farPlane": 1000.0,
          "isActive": true
        }
      }
    ],
    "activeCameraId": "UUID-STRING"
  },
  "version": "2.0",
  "tags": [],
  "notes": ""
}
```

## Entity Serialization

### Entity Data Structures
```swift
// ProjectManager.swift structures
struct ProjectSceneData: Codable {
    var entities: [ProjectEntityData] = []
    var activeCameraId: String?
}

struct ProjectEntityData: Codable {
    var id: UUID
    var name: String
    var type: String  // Entity type as string
    var transform: TransformData
    var lightData: LightData?
    var modelPath: String?
    var cameraData: CameraData?
}

struct CameraData: Codable {
    var fov: Float
    var nearPlane: Float
    var farPlane: Float
    var isActive: Bool
}
```

### Entity Serialization Pattern
```swift
// Convert Entity to serializable format
func serializeEntity(_ entity: any SceneEntity) -> ProjectEntityData {
    var data = ProjectEntityData(
        id: entity.id,
        name: entity.name,
        type: entity.entityType.rawValue,
        transform: TransformData(from: entity)
    )
    
    // Type-specific data
    if let camera = entity as? CameraEntity {
        data.cameraData = CameraData(
            fov: camera.fov,
            nearPlane: camera.nearPlane,
            farPlane: camera.farPlane,
            isActive: camera.isActive
        )
    } else if let light = entity as? LightEntity {
        data.lightData = LightData(from: light)
    } else if let model = entity as? ModelEntity {
        data.modelPath = model.modelURL?.path
    }
    
    return data
}
```

### Entity Deserialization Pattern
```swift
// Recreate Entity from saved data
func deserializeEntity(from data: ProjectEntityData) -> (any SceneEntity)? {
    guard let entityType = SceneEntityType(rawValue: data.type) else { return nil }
    
    switch entityType {
    case .camera:
        let camera = CameraEntity(name: data.name)
        camera.position = SIMD3<Float>(
            data.transform.position[0],
            data.transform.position[1],
            data.transform.position[2]
        )
        if let cameraData = data.cameraData {
            camera.fov = cameraData.fov
            camera.nearPlane = cameraData.nearPlane
            camera.farPlane = cameraData.farPlane
            camera.isActive = cameraData.isActive
        }
        return camera
        
    case .light:
        guard let lightData = data.lightData else { return nil }
        let lightType = LightEntity.LightType(rawValue: lightData.type) ?? .point
        let light = LightEntity(name: data.name, type: lightType)
        // Apply transform and light data...
        return light
        
    case .model:
        let model = ModelEntity(modelNamed: data.name)
        // Apply transform...
        if let modelPath = data.modelPath {
            Task {
                await model.load(from: URL(fileURLWithPath: modelPath))
            }
        }
        return model
        
    case .empty:
        let empty = Entity(name: data.name)
        // Apply transform...
        return empty
    }
}
```

## File Dialog Implementation (Updated)

### FileDocumentTypes.swift
```swift
// Document types for Entity system
struct ProjectDocument: FileDocument {
    static var readableContentTypes: [UTType] {
        [UTType(filenameExtension: "rvproject") ?? .json]
    }
    
    let projectManager: ProjectManager
    let sceneManager: SceneManager
    
    func fileWrapper(configuration: WriteConfiguration) throws -> FileWrapper {
        // Convert entities to ProjectSceneData
        let sceneData = ProjectSceneData(from: sceneManager)
        
        var project = projectManager.currentProject ?? ProjectFile(
            name: "Untitled",
            url: URL(fileURLWithPath: ""),
            dateCreated: Date(),
            dateModified: Date()
        )
        project.sceneData = sceneData
        project.dateModified = Date()
        
        let encoder = JSONEncoder()
        encoder.outputFormatting = .prettyPrinted
        encoder.dateEncodingStrategy = .iso8601
        let data = try encoder.encode(project)
        
        return FileWrapper(regularFileWithContents: data)
    }
}
```

## Import/Export with Entities

### Model Import Workflow
```swift
// SceneManager.swift
func importModel(from url: URL) async {
    let model = ModelEntity(modelNamed: url.lastPathComponent)
    
    // Start loading
    model.isLoading = true
    addEntity(model)
    
    // Load async
    await model.load(from: url)
    
    // Position at origin
    model.position = SIMD3<Float>(0, 0, 0)
    updateEntity(model)
}
```

### Export Formats
```swift
enum ExportFormat: String, CaseIterable, Codable {
    case json = "JSON"
    case usdz = "USDZ"
    case realityFile = "Reality"
    
    var fileExtension: String {
        switch self {
        case .json: return "json"
        case .usdz: return "usdz"
        case .realityFile: return "reality"
        }
    }
}

// Export implementation
func exportScene(format: ExportFormat) async throws {
    switch format {
    case .json:
        // Export entities as JSON
        let sceneData = ProjectSceneData(from: sceneManager)
        let data = try JSONEncoder().encode(sceneData)
        // Save data...
        
    case .usdz:
        // Convert entities to USDZ
        let exportEntity = RealityKit.Entity()
        for entity in sceneManager.entities {
            if let rkEntity = sceneManager.getRealityEntity(for: entity) {
                exportEntity.addChild(rkEntity.clone(recursive: true))
            }
        }
        // Export USDZ when API available...
        
    case .realityFile:
        // Similar to USDZ...
    }
}
```

## Security-Scoped Resources (Updated)

### Entity Model Loading
```swift
// ModelEntity.swift pattern
public func load(from url: URL) async {
    self.isLoading = true
    self.modelURL = url
    
    do {
        // Security scope
        let accessing = url.startAccessingSecurityScopedResource()
        defer {
            if accessing {
                url.stopAccessingSecurityScopedResource()
            }
        }
        
        // Load with RealityKit
        let rkModel = try await RealityKit.ModelEntity.loadModel(contentsOf: url)
        
        // Auto-scale
        let bounds = rkModel.model?.mesh.bounds.extents ?? [1, 1, 1]
        let maxDimension = max(bounds.x, bounds.y, bounds.z)
        let scaleFactor = maxDimension > 0 ? 2.0 / maxDimension : 1.0
        rkModel.scale = [scaleFactor, scaleFactor, scaleFactor]
        
        // Setup for interaction
        rkModel.generateCollisionShapes(recursive: true)
        rkModel.components.set(InputTargetComponent())
        
        // Update wrapper
        self.model = rkModel.model
        self.name = url.deletingPathExtension().lastPathComponent
        self.scale = rkModel.scale
        
    } catch {
        print("❌ Failed to load model: \(error)")
        // Create fallback cube...
    }
    
    self.isLoading = false
}
```

## ProjectManager Integration

### Save Operation (Entity-Based)
```swift
func saveProject(_ sceneManager: SceneManager) async throws {
    guard let project = currentProject else {
        throw ProjectError.noCurrentProject
    }
    
    var updatedProject = project
    updatedProject.sceneData = ProjectSceneData(from: sceneManager)
    updatedProject.dateModified = Date()
    
    let encoder = JSONEncoder()
    encoder.outputFormatting = .prettyPrinted
    let data = try encoder.encode(updatedProject)
    try data.write(to: project.url)
    
    await MainActor.run {
        self.currentProject = updatedProject
    }
}
```

### Load Operation (Entity-Based)
```swift
func loadProject(into sceneManager: SceneManager) async throws {
    guard let project = currentProject,
          let sceneData = project.sceneData else {
        throw ProjectError.noSceneData
    }
    
    await MainActor.run {
        // Clear existing scene
        sceneManager.clearScene()
        
        // Load entities
        for entityData in sceneData.entities {
            if let entity = createEntity(from: entityData) {
                sceneManager.addEntity(entity)
            }
        }
        
        // Set active camera
        if let activeCameraId = sceneData.activeCameraId,
           let cameraUUID = UUID(uuidString: activeCameraId),
           let camera = sceneManager.entity(withId: cameraUUID) as? CameraEntity {
            sceneManager.setActiveCamera(camera)
        }
    }
}
```

## Error Handling (Entity Context)

### Entity-Specific Errors
```swift
enum EntityFileError: LocalizedError {
    case invalidEntityType(String)
    case missingEntityData
    case corruptTransform
    case modelLoadFailed(URL)
    
    var errorDescription: String? {
        switch self {
        case .invalidEntityType(let type):
            return "Unknown entity type: \(type)"
        case .missingEntityData:
            return "Entity data is incomplete"
        case .corruptTransform:
            return "Entity transform data is corrupted"
        case .modelLoadFailed(let url):
            return "Failed to load model from: \(url.lastPathComponent)"
        }
    }
}
```

## Platform-Specific Handling

### ContentView.swift Pattern
```swift
#if !os(tvOS)
.fileImporter(
    isPresented: $isShowingFilePicker,
    allowedContentTypes: [
        UTType(filenameExtension: "usdz") ?? .data,
        UTType(filenameExtension: "reality") ?? .data
    ],
    allowsMultipleSelection: false
) { result in
    handleFileSelection(result)
}
#endif
```

## Best Practices (Entity System)

### DO: Type-Check Entity Operations
```swift
// Correct: Safe entity casting
if let model = entity as? ModelEntity {
    await model.load(from: url)
}
```

### DO: Handle Async Model Loading
```swift
// Correct: Non-blocking load
Task {
    await sceneManager.importModel(from: url)
}
```

### DO: Preserve Entity Relationships
```swift
// Correct: Maintain parent-child
for entity in sceneData.entities {
    if let parentId = entityData.parentId {
        // Restore hierarchy
    }
}
```

### DON'T: Mix Node and Entity
```swift
// Wrong: Node system is removed
sceneManager.nodes.append(node)  // ❌

// Correct: Use Entity system
sceneManager.addEntity(entity)   // ✅
```

## Performance Optimizations

### Entity-Specific
- **Async Model Loading**: ModelEntity loads asynchronously
- **Lazy Component Access**: Only serialize needed components
- **Batch Entity Updates**: Group multiple entity changes
- **Efficient Serialization**: Minimal data per entity

### Memory Management
```swift
// Auto-release loaded models
autoreleasepool {
    for url in modelURLs {
        let model = ModelEntity()
        await model.load(from: url)
        sceneManager.addEntity(model)
    }
}
```

## Migration Notes (Node → Entity)

### What Changed
| Old (Node) | New (Entity) |
|------------|--------------|
| `NodeData` | `ProjectEntityData` |
| `node.nodeType` | `entity.entityType` |
| `SceneNode` serialization | `SceneEntity` serialization |
| `ModelNode.loadModel()` | `ModelEntity.load()` |
| Sync loading | Async loading |

### Compatibility
- Old .rvproject files with Node data won't load
- Manual migration required for old projects
- New format is cleaner and more efficient

## Testing Checklist

- [x] Save project with entities
- [x] Load project with entities
- [x] Import USDZ models
- [x] Import Reality files
- [x] Security scope handling
- [x] Error recovery
- [x] Cross-platform dialogs
- [ ] Export to USDZ (pending API)
- [ ] Export to Reality (pending API)

## Future Enhancements

- [ ] Project migration tool (Node → Entity)
- [ ] Incremental saves for large projects
- [ ] Asset dependency tracking
- [ ] Cloud sync with entity data
- [ ] Collaborative editing support
- [ ] Version control integration

## Summary

File operations are fully updated for the Entity/ECS architecture:
- ✅ Entity serialization/deserialization
- ✅ Async model loading in entities
- ✅ Security-scoped resource handling
- ✅ Cross-platform file dialogs
- ✅ ProjectSceneData with entities

The system maintains backward compatibility where possible while fully embracing the new Entity architecture.