# Android App Style Guide

Standards for Android applications.

## File Structure

### Top-Level Structure
```
project_name/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/
│   │   │   │   └── com/
│   │   │   │       └── example/
│   │   │   │           └── app/
│   │   │   │               ├── MainActivity.kt
│   │   │   │               ├── ui/
│   │   │   │               │   ├── theme/
│   │   │   │               │   └── screens/
│   │   │   │               ├── data/
│   │   │   │               │   ├── repository/
│   │   │   │               │   └── model/
│   │   │   │               ├── domain/
│   │   │   │               │   └── usecase/
│   │   │   │               └── di/
│   │   │   │           build.gradle.kts
│   │   │   ├── res/
│   │   │   │   ├── layout/
│   │   │   │   ├── values/
│   │   │   │   ├── drawable/
│   │   │   │   └── mipmap-*/
│   │   │   └── AndroidManifest.xml
│   │   └── build.gradle.kts
├── gradle/
│   └── wrapper/
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
└── README.md
```

### Package Organization
- Use feature-based or layer-based organization:
  - `ui/` - Activities, Fragments, ViewModels
  - `data/` - Repositories, Data Sources, Models
  - `domain/` - Use Cases, Business Logic
  - `di/` - Dependency Injection

## Formatting

- **Language**: Kotlin (preferred), Java (legacy)
- **Indent**: 4 spaces (no tabs)
- **Max line length**: 100 chars
- **Line endings**: LF (Unix-style)
- **Braces**: K&R style

## Naming Conventions

| Type | Convention | Example |
|:-----|:-----------|:--------|
| Activities | `<Feature>Activity` | `MainActivity`, `SettingsActivity` |
| Fragments | `<Feature>Fragment` | `HomeFragment`, `ProfileFragment` |
| ViewModels | `<Feature>ViewModel` | `HomeViewModel` |
| Layouts | `activity_<name>.xml` | `activity_main.xml` |
| Drawables | `<type>_<name>` | `ic_github`, `bg_card` |
| Colors | `color_<name>` | `color_primary`, `color_surface` |
| Strings | `<prefix>_<name>` | `btn_submit`, `msg_error` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |

## Code Organization

### Architecture
- Use MVVM or Clean Architecture
- Separate UI, Business Logic, and Data layers
- Use Dependency Injection (Hilt recommended)

### Key Principles
- Single Responsibility: One class, one purpose
- Dependency Inversion: Depend on abstractions, not concretions
- Keep classes focused and small

## UI Development

### Jetpack Compose
- Use Compose for new UI development
- Follow Material Design 3 guidelines
- Use remember and rememberSaveable for state

```kotlin
@Composable
fun MyScreen(viewModel: MyViewModel = hiltViewModel()) {
    val state by viewModel.uiState.collectAsState()

    Column {
        Text(state.message)
        Button(onClick = { viewModel.onAction() }) {
            Text("Submit")
        }
    }
}
```

### Views (XML)
- Use ConstraintLayout for complex layouts
- Use RecyclerView for lists
- Follow naming conventions for IDs

## Async Operations

### Coroutines
- Use Kotlin Coroutines for async work
- Use ViewModelScope for UI-related operations
- Use lifecycleScope for lifecycle-aware operations

```kotlin
class MyViewModel : ViewModel() {
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState

    fun loadData() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            val data = repository.getData()
            _uiState.value = UiState.Success(data)
        }
    }
}
```

### Threading
- Do not perform heavy operations on main thread
- Use Dispatchers.IO for network/disk operations
- Use Dispatchers.Main for UI updates

## Preferred Libraries

| Purpose | Library |
|:--------|:--------|
| DI | Hilt |
| Networking | Retrofit, OkHttp |
| JSON | Kotlinx Serialization |
| Image Loading | Coil |
| Database | Room |
| Navigation | Navigation Compose |
| Async | Kotlin Coroutines |
| UI | Jetpack Compose |

## Common Mistakes and Correct Implementation

### 1. DON'T: Perform network operations on main thread

```kotlin
// WRONG - blocks UI
val response = api.call()
```

### ✅ DO: Use coroutines with IO dispatcher

```kotlin
// CORRECT - non-blocking
viewModelScope.launch(Dispatchers.IO) {
    val response = api.call()
    withContext(Dispatchers.Main) {
        // Update UI
    }
}
```

### 2. DON'T: Hardcode strings

```kotlin
// WRONG
Text("Submit")
```

### ✅ DO: Use string resources

```kotlin
// CORRECT
Text(stringResource(R.string.btn_submit))
```

### 3. DON'T: Leak context in ViewModel

```kotlin
// WRONG
class MyViewModel(val app: Application) { }
```

### ✅ DO: Use application with Hilt

```kotlin
// CORRECT
@HiltViewModel
class MyViewModel @Inject constructor(
    private val repo: MyRepository
) : ViewModel()
```

### 4. DON'T: Use lateinit var for state

```kotlin
// WRONG
lateinit var data: String
```

### ✅ DO: Use StateFlow

```kotlin
// CORRECT
private val _data = MutableStateFlow("")
val data: StateFlow<String> = _data.asStateFlow()
```