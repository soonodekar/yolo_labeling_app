# YOLO Labeling App - Architecture Overview

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                          UI LAYER                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Project    │  │  Annotation  │  │   Review     │          │
│  │  ViewModel   │  │  ViewModel   │  │  ViewModel   │          │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │
│         │                  │                  │                   │
│         └──────────────────┼──────────────────┘                   │
│                            │                                      │
│  ┌──────────────┐          │                                      │
│  │    Label     │          │                                      │
│  │  ViewModel   │          │                                      │
│  └──────┬───────┘          │                                      │
│         │                  │                                      │
└─────────┼──────────────────┼──────────────────────────────────────┘
          │                  │
┌─────────┼──────────────────┼──────────────────────────────────────┐
│         │    REPOSITORY LAYER          │                          │
├─────────┼──────────────────┼──────────────────────────────────────┤
│         ▼                  ▼                  ▼                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Project    │  │    Label     │  │  Annotation  │          │
│  │  Repository  │  │  Repository  │  │  Repository  │          │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │
│         │                  │                  │                   │
│         └──────────────────┼──────────────────┘                   │
│                            │                                      │
└────────────────────────────┼──────────────────────────────────────┘
                             │
┌────────────────────────────┼──────────────────────────────────────┐
│         DATA ACCESS LAYER   │                                      │
├────────────────────────────┼──────────────────────────────────────┤
│         ┌──────────────────┴──────────────────┐                  │
│         │                                      │                  │
│         ▼                  ▼                  ▼                   │
│  ┌──────────┐      ┌──────────┐      ┌──────────────┐           │
│  │ Project  │      │  Label   │      │ImageAnnotation│           │
│  │   DAO    │      │   DAO    │      │     DAO       │           │
│  └──────────┘      └──────────┘      └───────────────┘           │
│                                                                   │
│         ┌───────────────────┐                                     │
│         │  BoundingBox DAO  │                                     │
│         └─────────┬─────────┘                                     │
│                   │                                               │
└───────────────────┼───────────────────────────────────────────────┘
                    │
┌───────────────────┼───────────────────────────────────────────────┐
│    DATABASE LAYER │                                               │
├───────────────────┼───────────────────────────────────────────────┤
│                   ▼                                               │
│         ┌─────────────────────┐                                   │
│         │  Room Database      │                                   │
│         │                     │                                   │
│         │  ┌───────────────┐ │                                   │
│         │  │   Projects    │ │                                   │
│         │  ├───────────────┤ │                                   │
│         │  │    Labels     │ │                                   │
│         │  ├───────────────┤ │                                   │
│         │  │ImageAnnotations│                                   │
│         │  ├───────────────┤ │                                   │
│         │  │ BoundingBoxes │ │                                   │
│         │  └───────────────┘ │                                   │
│         └─────────────────────┘                                   │
└───────────────────────────────────────────────────────────────────┘
```

## Data Flow Examples

### Creating a New Annotation

```
User Action (Capture Image)
    ↓
AnnotationViewModel.createAnnotation()
    ↓
AnnotationRepository.createAnnotation()
    ↓
ImageAnnotationDao.insert()
    ↓
Room Database (SQLite)
    ↓
Flow emits new data
    ↓
ViewModel updates UI State
    ↓
UI updates automatically
```

### Adding Bounding Boxes

```
User draws box on screen
    ↓
AnnotationViewModel.addBoundingBox(pixel coordinates)
    ↓
Convert to normalized YOLO format
    ↓
AnnotationRepository.createBoundingBox()
    ↓
BoundingBoxDao.insert()
    ↓
Database updated
    ↓
Flow emits updated CompleteAnnotation
    ↓
UI shows new bounding box
```

## Database Relationships

```
┌─────────────┐
│   Project   │
│   id (PK)   │
└──────┬──────┘
       │ 1
       │
       │ N
       ├──────────────────┐
       │                  │
       ▼                  ▼
┌──────────────┐   ┌─────────────────┐
│    Label     │   │ ImageAnnotation │
│   id (PK)    │   │    id (PK)      │
│ projectId(FK)│   │  projectId(FK)  │
│  classIndex  │   └────────┬────────┘
│   colorHex   │            │ 1
└──────┬───────┘            │
       │ 1                  │ N
       │                    │
       │                    ▼
       │            ┌───────────────┐
       │            │ BoundingBox   │
       └───────────▶│   id (PK)     │
              N     │annotationId(FK)│
                    │  labelId (FK)  │
                    │   centerX      │
                    │   centerY      │
                    │    width       │
                    │   height       │
                    └────────────────┘
```

## YOLO Format Conversion

### Normalized Coordinates (Database Storage)
```
centerX = (left + right) / 2 / imageWidth
centerY = (top + bottom) / 2 / imageHeight
width = (right - left) / imageWidth
height = (bottom - top) / imageHeight

All values: 0.0 to 1.0
```

### Pixel Coordinates (UI Display)
```
left = (centerX - width/2) * imageWidth
top = (centerY - height/2) * imageHeight
right = (centerX + width/2) * imageWidth
bottom = (centerY + height/2) * imageHeight
```

### YOLO .txt Export Format
```
<class_index> <center_x> <center_y> <width> <height>

Example:
0 0.5 0.5 0.3 0.4
1 0.2 0.3 0.15 0.2
```

## ViewModel State Management

### ProjectViewModel States
```kotlin
sealed class ProjectUiState {
    Loading          // Fetching data
    Empty            // No projects
    Success          // Projects loaded
    Error            // Operation failed
    ProjectCreated   // New project added
    ProjectUpdated   // Project modified
    ProjectDeleted   // Project removed
    StatsLoaded      // Statistics ready
}
```

### AnnotationViewModel States
```kotlin
sealed class AnnotationUiState {
    Idle             // Ready for action
    Loading          // Processing
    AnnotationLoaded // Data ready
    AnnotationCreated // New annotation
    AnnotationSaved  // Complete save
    AnnotationDeleted // Removed
    BoxAdded         // New box
    BoxUpdated       // Box modified
    BoxDeleted       // Box removed
    AllBoxesDeleted  // All boxes cleared
    Error            // Operation failed
}
```

## Key Design Patterns

1. **Repository Pattern**: Abstracts data sources
2. **MVVM**: Clear separation of concerns
3. **Reactive Programming**: Flow-based updates
4. **Single Source of Truth**: Database is authority
5. **Result Type**: Explicit error handling
6. **Dependency Injection**: Hilt manages dependencies

## Performance Considerations

- **Flow**: Reactive updates without manual refreshes
- **Pagination**: Load images in chunks
- **Cascade Delete**: Database handles relationships
- **Normalized Storage**: Efficient coordinate storage
- **Indexed Queries**: Fast lookups on foreign keys
- **Coroutines**: Non-blocking database operations
