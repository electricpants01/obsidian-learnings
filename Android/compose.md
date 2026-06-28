# Jetpack Compose — Patterns in the Modularization Branch

## Key Takeaways

1. **Every screen is a `@Composable` function** — no Fragments, no XML layouts.
2. **ViewModel is obtained via `hiltViewModel()`** as a default argument to the Route composable.
3. **State is collected with `collectAsState()`** — not `collectAsStateWithLifecycle()`.
4. **Screen decomposition**: `XxxRoute()` → `XxxScreen()` → `Content()`.
5. **`sealed interface` for UI state** — every feature defines `Loading` + `Success(data)`.
6. **Material 3** — `Scaffold`, `CenterAlignedTopAppBar`, `ListItem`, `Checkbox`, `CircularProgressIndicator`.
7. **No previews, no side effects** — this branch is intentionally minimal.

---

## Screen Pattern (ListRoute)

```kotlin
@Composable
fun ListRoute(
    onGoToItem: (Long) -> Unit,
    modifier: Modifier = Modifier,
    viewModel: ListViewModel = hiltViewModel()  // ← Hilt-injected, default arg
) {
    val state by viewModel.uiState.collectAsState()  // ← collects StateFlow
    ListScreen(state, onGoToItem, viewModel::bookmark, modifier)
}

@Composable
internal fun ListScreen(
    state: ListUiState,
    onGoToItem: (Long) -> Unit,
    onBookmarkItem: (Long, Boolean) -> Unit,
    modifier: Modifier = Modifier
) {
    when (state) {
        ListUiState.Loading -> Loading(modifier)           // ← reused from :core:ui
        is ListUiState.Success -> Content(state.data, ...)  // ← private composable
    }
}
```

### Why this pattern?

| Aspect | Purpose |
|---|---|
| `Route` composable | Public entry point — calls `hiltViewModel()`, collects state |
| `Screen` composable | Pure function — routes state to `Loading` or `Content` |
| `Content` composable | Private — actual UI layout, no ViewModel knowledge |
| `internal` visibility | Only `Route` is public; `Screen` and `Content` are module-internal |

---

## State Collection

The canonical one-liner:

```kotlin
val state by viewModel.uiState.collectAsState()
```

- `by` = property delegation, unwraps `State<T>` to `T`
- `collectAsState()` = collects `StateFlow<T>` into a Compose `State<T>`
- This project **does not** use `collectAsStateWithLifecycle()` — the `multimodule` branch uses the simpler version

---

## State Rendering (when block)

```kotlin
when (state) {
    ListUiState.Loading -> Loading(modifier)
    is ListUiState.Success -> Content(state.data, onGoToItem, onBookmarkItem, modifier)
}
```

The `sealed interface` ensures exhaustive `when` — the compiler catches missing branches.

---

## Details Feature (Top App Bar)

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
internal fun Content(
    state: DetailsUiState.Success,
    onGoBack: () -> Unit,
    modifier: Modifier = Modifier
) {
    Scaffold(
        topBar = {
            CenterAlignedTopAppBar(
                title = { Text(state.title) },
                navigationIcon = {
                    IconButton(onGoBack, modifier = Modifier.testTag("nav_icon")) {
                        Icon(Icons.Default.ArrowBack, null)
                    }
                }
            )
        },
        modifier = modifier
    ) { padding ->
        Text(state.description, Modifier.padding(padding))
    }
}
```

**Key points:**
- `Scaffold` provides the app bar + content area with correct insets
- `testTag("nav_icon")` enables UI tests to find the back button
- `Icons.Default.ArrowBack` from `material-icons-extended`

---

## List Feature (LazyColumn + Material 3 ListItem)

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
internal fun Content(
    items: List<MyModelUiState>,
    onGoToItem: (Long) -> Unit,
    onBookmarkItem: (Long, Boolean) -> Unit,
    modifier: Modifier = Modifier
) {
    LazyColumn(modifier = modifier.fillMaxSize()) {
        itemsIndexed(items = items) { index, item ->
            ListItem(
                headlineText = { Text(text = item.title) },
                trailingContent = { Text(text = item.date) },
                leadingContent = {
                    Checkbox(
                        checked = item.isBookmarked,
                        onCheckedChange = { isChecked ->
                            onBookmarkItem(item.id, isChecked)
                        }
                    )
                },
                modifier = Modifier
                    .clickable { onGoToItem(item.id) }
                    .testTag("item_$index")
            )
        }
    }
}
```

**Key points:**
- `ListItem` from Material 3 — structured row with headline, trailing, leading slots
- `itemsIndexed` provides both the item and its index (for `testTag`)
- `testTag("item_$index")` — unique identifiers per row for UI tests
- Callbacks flow up: `onBookmarkItem` → `viewModel::bookmark`, `onGoToItem` → navigation

---

## Compose Gradle Setup

```kotlin
// build-logic/convention/.../Compose.kt
fun Project.configureCompose() {
    buildFeatures { compose = true }

    composeOptions {
        kotlinCompilerExtensionVersion = libs.findVersion("composeCompiler").get().toString()
    }

    dependencies {
        val composeBom = platform(libs.androidx.compose.bom)
        implementation(composeBom)
        implementation(libs.androidx.compose.foundation.core)
        implementation(libs.androidx.compose.foundation.layout)
        implementation(libs.androidx.compose.material3)
        implementation(libs.androidx.compose.material.iconsExtended)
        implementation(libs.androidx.compose.ui.tooling.preview)
        implementation(libs.androidx.compose.ui.test.manifest)
        implementation(libs.androidx.hilt.navigation.compose)
    }
}
```

The `ComposeConventionPlugin` applies `configureCompose()` — any module that needs Compose just applies the `sample.compose` plugin.

---

## Theming

`MyAppTheme.kt` in `:app:mobile` wraps `MaterialTheme` with light/dark color schemes. All feature composables render inside this theme. Features don't set their own theme — they inherit from the activity-level wrapper.

---

## Wear Compose

The wear app uses:
- `SwipeDismissableNavHost` + `rememberSwipeDismissableNavController()`
- `ScalingLazyColumn` instead of `LazyColumn`
- `Chip` instead of `ListItem`
- Wear-specific Material theme

Same architectural split: `HomeRoute` → `HomeScreen` → `Content`.
