# YOLO v8 Labeling App - Data Layer Implementation

This is a complete data layer implementation for an Android YOLO v8 labeling application using MVVM architecture.

## 📁 Project Structure

```
yolo-labeling-app/
├── data/
│   ├── model/                      # Data entities
│   │   ├── Project.kt             # Project entity
│   │   ├── Label.kt               # Label entity with class index
│   │   ├── ImageAnnotation.kt     # Image metadata
│   │   ├── BoundingBox.kt         # Bounding box with YOLO format
│   │   └── Relations.kt           # Room relationships
│   ├── local/
│   │   ├── dao/                   # Data Access Objects
│   │   │   ├── ProjectDao.kt
│   │   │   ├── LabelDao.kt
│   │   │   ├── ImageAnnotationDao.kt
│   │   │   └── BoundingBoxDao.kt
│   │   └── database/
│   │       └── YoloLabelingDatabase.kt
│   └── repository/                # Repository layer
│       ├── ProjectRepository.kt
│       ├── LabelRepository.kt
│       └── AnnotationRepository.kt
└── ui/
    ├── project/
    │   ├── ProjectViewModel.kt    # Project management
    │   └── LabelViewModel.kt      # Label management
    ├── annotate/
    │   └── AnnotationViewModel.kt # Annotation creation/editing
    └── review/
        └── ReviewViewModel.kt     # Review & browse annotations
```

## 🗄️ Database Schema

### Tables

**projects**
- id (PK)
- name
- description
- createdAt
- updatedAt

**labels**
- id (PK)
- projectId (FK → projects.id)
- name
- colorHex (e.g., "#FF5733")
- classIndex (YOLO class index: 0, 1, 2, ...)

**image_annotations**
- id (PK)
- projectId (FK → projects.id)
- imagePath (absolute path)
- imageWidth
- imageHeight
- createdAt
- updatedAt

**bounding_boxes**
- id (PK)
- annotationId (FK → image_annotations.id)
- labelId (FK → labels.id)
- centerX (normalized 0-1)
- centerY (normalized 0-1)
- width (normalized 0-1)
- height (normalized 0-1)

All relationships use CASCADE delete.

## 🎯 Key Features

### 1. Multi-Project Support
- Each project has its own set of labels
- Labels have unique class indices per project
- Projects are isolated with cascade deletion

### 2. YOLO Format Support
- Bounding boxes stored in normalized YOLO format (center_x, center_y, width, height)
- Helper methods for pixel ↔ normalized coordinate conversion
- Ready for YOLO .txt file export

### 3. Repository Pattern
- Clean separation of concerns
- Result type for error handling
- Flow-based reactive updates
- Comprehensive CRUD operations

### 4. ViewModels
- **ProjectViewModel**: Create, update, delete projects; manage statistics
- **LabelViewModel**: Manage labels with auto-indexing and validation
- **AnnotationViewModel**: Create/edit annotations with bounding boxes
- **ReviewViewModel**: Browse, filter, and review annotated images

### 5. Data Relationships
- `ProjectWithLabels`: Project + all labels
- `AnnotationWithBoxes`: Image + all bounding boxes
- `BoundingBoxWithLabel`: Box + label information
- `CompleteAnnotation`: Image + boxes + labels (full data)

## 🔧 Usage Examples

### Creating a Project with Labels
```kotlin
// In your Activity/Fragment
viewModel.createProject(
    name = "Face Detection",
    description = "Training data for face detection model",
    labels = listOf("face", "eye", "nose", "mouth")
)

// Observe state
viewModel.uiState.collect { state ->
    when (state) {
        is ProjectUiState.ProjectCreated -> {
            // Navigate to annotation screen
            navigateToAnnotation(state.projectId)
        }
        is ProjectUiState.Error -> {
            showError(state.message)
        }
    }
}
```

### Adding Bounding Boxes
```kotlin
// Create annotation for new image
annotationViewModel.createAnnotation(
    projectId = currentProjectId,
    imagePath = capturedImagePath,
    imageWidth = 1920,
    imageHeight = 1080
)

// Add bounding box
annotationViewModel.addBoundingBox(
    labelId = selectedLabelId,
    left = 100,
    top = 200,
    right = 300,
    bottom = 400
)
```

### Reviewing Annotations
```kotlin
// Load all annotations
reviewViewModel.loadAnnotations(projectId)

// Navigate through images
reviewViewModel.goToNext()
reviewViewModel.goToPrevious()

// Filter by label
reviewViewModel.filterByLabel(labelId)

// Show only unannotated
reviewViewModel.toggleShowOnlyUnannotated()
```

## 📦 Dependencies

Key dependencies required:
```kotlin
// Room Database
implementation("androidx.room:room-runtime:2.6.1")
implementation("androidx.room:room-ktx:2.6.1")
kapt("androidx.room:room-compiler:2.6.1")

// Coroutines
implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")

// Lifecycle & ViewModel
implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0")

// Hilt DI
implementation("com.google.dagger:hilt-android:2.48.1")
kapt("com.google.dagger:hilt-compiler:2.48.1")
```

## 🚀 Next Steps

To complete the application, you'll need to implement:

1. **UI Layer**
   - Project list/creation screens
   - Camera capture with overlay
   - Annotation editor with touch handling
   - Review gallery

2. **Custom Views**
   - `BoundingBoxOverlayView` for drawing/editing boxes
   - `LabelSelectorView` for quick label selection

3. **YOLO Export**
   - Generate .txt files with annotations
   - Create dataset.yaml with class names
   - ZIP export functionality

4. **Camera Integration**
   - CameraX setup
   - Real-time preview with overlay
   - Image capture

Would you like me to implement any of these next components?

## 📝 Notes

- All normalized coordinates are in range [0, 1]
- Image paths should be absolute paths in app storage
- Class indices must be unique per project
- Label names must be unique per project
- Cascade delete removes all child records
