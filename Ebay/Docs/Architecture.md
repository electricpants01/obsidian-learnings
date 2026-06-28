# Architecture

This doc covers how the general architecture patterns are applied specifically in Digital Collections. For the full architecture guide (layering, UDF, testing best practices), see [Collectibles Architecture & Best Practices](../docs/CollectiblesArchitectureAndBestPractices.md).

---

## Screen Anatomy

Every screen in Digital Collections follows a consistent structure:

```
┌─ view/<screenName>/
│   ├── <ScreenName>Screen.kt        # Composable UI
│   ├── <ScreenName>ViewModel.kt     # State + event handling
│   ├── <ScreenName>Navigation.kt    # Route class + navigateTo + NavGraphBuilder
│   └── <ScreenName>TrackingAssets.kt # Tracking constants
```

### Standard Flow

```
User Action → Event (sealed interface) → ViewModel.handleEvent()
  → UseCase / Repository → Data Source (GraphQL / REST)
  → UiState update → Compose recomposition
  → Effect (one-shot: navigation, snackbar, tracking)
```

---

## CollectiblesScaffold Delegate

`CollectiblesScaffold` is a Kotlin delegate pattern that standardizes screen state management. Each ViewModel delegates to it:

```kotlin
class MyScreenViewModel @Inject constructor(
    private val scaffold: CollectiblesScaffold<MyUiState, MyEvent, MyEffect>
) : CollectiblesScaffold<MyUiState, MyEvent, MyEffect> by scaffold {

    init {
        scaffold.initialize()
    }
}
```

The scaffold provides:
- **State** — `StateFlow<MyUiState>` for Compose collection
- **Events** — `handleEvent(event: MyEvent)` for user actions
- **Effects** — `SharedFlow<MyEffect>` for one-shot side effects
- **InitContent + Refresh** — Standard initialization and pull-to-refresh

### State vs Effects

| | State | Effects |
|---|---|---|
| **Scope** | Screen-level, survives recomposition | One-shot, consumed once |
| **Examples** | `isLoading`, `items`, `selectedTab` | Navigation, snackbar, tracking |
| **Collection** | `collectAsStateWithLifecycle()` | `LaunchedEffect` + `collect()` |

---

## Activity-Level State

`CollectibleActivityViewModel` manages state shared across all screens in the activity:

- **Action Bar** — Title, subtitle, navigation icon, action buttons
- **Options Menu** — Overflow menu items
- **Bottom Sheet** — App-level bottom sheets (add item options, quick edit)
- **Snackbar** — Global snackbar messages
- **Bottom Navigation** — Tab state for the hub screen

Screens communicate with it by posting state updates via `CollectiblesScaffold`.

---

## InitContent + Refresh Pattern

Every screen follows a two-phase initialization:

```kotlin
// Navigation.kt
fun NavGraphBuilder.myScreen() {
    composable<MyRoute> {
        val viewModel = viewModel<MyViewModel>(factory = ...)
        MyScreen(viewModel)
    }
}

// ViewModel
init {
    scaffold.initialize(
        initContent = { loadInitialData() },
        onRefresh = { refreshData() }
    )
}
```

---

## ActionNavigationTarget

Cross-feature navigation (e.g., from View Item to Digital Collections) uses `ActionNavigationTarget`:

```kotlin
class DigitalCollectionsNavigationTarget @Inject constructor(
    private val digitalCollectionsFactory: DigitalCollectionsFactory
) : ActionNavigationTarget {

    override val action: NavigationAction = ACTION_DIGITAL_COLLECTIONS

    override fun createIntent(context: Context, params: Map<String, String>): Intent {
        val page = params[PARAM_DIGITAL_COLLECTIONS_PAGE]
        return digitalCollectionsFactory.buildActivityIntent(context, page, params)
    }
}
```

This is registered in `DigitalCollectionsApplicationModule` and dispatched by the navigation framework.

---

## Feature Toggle Integration

Features are gated by toggles from `DigitalCollectionsFeatureToggles`. The pattern:

```kotlin
@Inject
lateinit var toggleRouter: ToggleRouter

fun isFeatureEnabled(): Boolean =
    toggleRouter.isEnabled(DigitalCollectionsFeatureToggles.MY_TOGGLE)
```

For testing, use `TestToggleRouter` to override specific toggles. See [Feature Toggles](FeatureToggles.md).

---

## Self-Contained Components

Some UI sections follow a factory pattern for reusability. These are documented in [Self-Contained Components](SelfContainedComponents.md). Current components:

- `UnifiedPriceGuidanceFactory` — Market data, price insights tabs
- `ShowGradedCardFactory` — Graded card / Card Insights display
- `ConditionSelectionFactory` — Condition selector (graded/ungraded)
- `PhotoCarouselFactory` — Image carousel for item photos
