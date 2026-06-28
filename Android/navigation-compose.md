# Navigation Compose — Patterns in the Modularization Branch

## Key Takeaways

1. **Single-Activity** — `MainActivity` hosts the entire `NavHost`.
2. **`NavHost` with `composable()` blocks** — each screen is a route.
3. **Two destinations** — `"list"` and `"details/{id}"`.
4. **Arguments via URL patterns** — `{id}` path parameter with `NavType.LongType`.
5. **`SavedStateHandle` extracts args** — DetailsViewModel reads `id` reactively.
6. **No `NavigationActions` wrapper, no drawer** — minimal navigation, direct `navController` usage.
7. **Wear uses `SwipeDismissableNavController`** — single `"home"` destination.

---

## Files to Check

| File | Module | Purpose |
|---|---|---|
| `MainActivity.kt` | `:app:mobile` | Single activity, sets `MyAppTheme { MainNavigation() }` |
| `Navigation.kt` | `:app:mobile` | `NavHost` + `composable()` destinations + `MainNavigation` |
| `ListRoute.kt` | `:feature:list` | `onGoToItem` callback triggers navigation |
| `DetailsRoute.kt` | `:feature:details` | `onGoBack` callback calls `popBackStack()` |
| `DetailsViewModel.kt` | `:feature:details` | Extracts `id` from `SavedStateHandle.getStateFlow()` |
| `Navigation.kt` (wear) | `:app:wear` | `SwipeDismissableNavHost` with single `"home"` route |

---

## Mobile Navigation Graph

```kotlin
@Composable
fun MainNavigation(
    navController: NavHostController = rememberNavController()
) {
    NavHost(navController = navController, startDestination = "list") {
        composable("list") {
            ListRoute(
                onGoToItem = { id ->
                    navController.navigate("details/$id")
                }
            )
        }

        composable(
            "details/{id}",
            listOf(navArgument("id") { type = NavType.LongType })
        ) {
            DetailsRoute(
                onGoBack = {
                    navController.popBackStack()
                }
            )
        }
    }
}
```

### Route Table

| Route | Arguments | Destination | Module |
|---|---|---|---|
| `"list"` | (none) | `ListRoute(onGoToItem)` | `:feature:list` |
| `"details/{id}"` | `id: Long` (LongType) | `DetailsRoute(onGoBack)` | `:feature:details` |

### What happens when the user taps an item:

1. `ListRoute` → `onGoToItem(item.id)` callback
2. `navController.navigate("details/$id")` — string interpolation builds the URL
3. Navigation Compose parses `{id}` from the URL and makes it available
4. `DetailsViewModel` reads it via `SavedStateHandle.getStateFlow<Long?>("id", null)`

### Key APIs:

| API | Purpose |
|---|---|
| `rememberNavController()` | Creates and remembers the `NavHostController` instance |
| `NavHost(startDestination = "list")` | The navigation container |
| `composable("route")` | Declares a destination as a `@Composable` function |
| `navArgument("id") { type = NavType.LongType }` | Declares a typed URL parameter |
| `navController.navigate("details/$id")` | Navigates to the details route with the item ID |
| `navController.popBackStack()` | Goes back to the previous destination |

---

## Argument Extraction: Two Approaches

### Approach 1: From `NavBackStackEntry` (manual)

```kotlin
composable("details/{id}", listOf(navArgument("id") { type = NavType.LongType })) { entry ->
    val id = entry.arguments?.getLong("id")    // ← manual extraction
    // pass id to screen
}
```

### Approach 2: From `SavedStateHandle` (used here)

```kotlin
// DetailsViewModel.kt
savedStateHandle.getStateFlow<Long?>("id", null)
    .filterNotNull()
    .flatMapLatest { id -> repository.observeModelById(id) }
```

**Approach 2 is better because:**
- Survives process death (the argument is stored in the saved state)
- Reactive — works as a `Flow`, reactively switching when `id` changes
- No manual parsing in the Composable
- The ViewModel owns the argument extraction, keeping the Composable clean

---

## Navigation Callback Pattern

The Compose screen receives navigation actions as **lambda callbacks**, not as
a `NavController` reference:

```kotlin
// ✅ Correct — callbacks, no NavController dependency
@Composable
fun ListRoute(onGoToItem: (Long) -> Unit, ...) { }

@Composable
fun DetailsRoute(onGoBack: () -> Unit, ...) { }

// In NavHost:
ListRoute(onGoToItem = { id -> navController.navigate("details/$id") })
DetailsRoute(onGoBack = { navController.popBackStack() })
```

### Why not pass `NavController` directly?

- **Decoupling** — the feature module doesn't depend on `:navigation-compose`
- **Testability** — tests can pass no-op lambdas without a real NavController
- **Single responsibility** — the navigation graph (in `:app:mobile`) owns routing logic

This is the **navigation callback pattern**: the app module wires routes → screens,
and screens only know about callbacks.

---

## No `popUpTo`, No `launchSingleTop`

Unlike the `main` branch, this branch:
- Does **not** use `popUpTo` / `launchSingleTop` / `restoreState`
- Does **not** have a `NavigationActions` wrapper class
- Does **not** use navigation results via URL arguments (like `?userMessage={userMessage}`)
- Does **not** have a drawer

Navigation is intentionally minimal: navigate forward, pop back. That's it.

---

## Wear Navigation

```kotlin
@Composable
fun MainNavigation(
    navController: NavHostController = rememberSwipeDismissableNavController()
) {
    SwipeDismissableNavHost(
        navController = navController,
        startDestination = "home"
    ) {
        composable("home") {
            HomeRoute()
        }
    }
}
```

### Differences from mobile:

| Aspect | Mobile | Wear |
|---|---|---|
| NavController factory | `rememberNavController()` | `rememberSwipeDismissableNavController()` |
| NavHost type | `NavHost` | `SwipeDismissableNavHost` |
| Destinations | 2 (`list`, `details/{id}`) | 1 (`home`) |
| Navigation action | Forward + back | Single screen, no navigation |

`SwipeDismissableNavController` enables the swipe-to-dismiss gesture on Wear OS
(navigate back by swiping left-to-right).

---

## Navigation vs Module Dependencies

```
:app:mobile  ──depends-on──►  :feature:list
  │   wires: ListRoute(onGoToItem = navigate)            │
  │                                                      │
  └──depends-on──►  :feature:details                     │
      wires: DetailsRoute(onGoBack = popBackStack)        │

:feature:list      does NOT depend on :feature:details
:feature:details   does NOT depend on :feature:list
Both              do NOT depend on Navigation Compose
```

Feature modules know nothing about each other or about navigation — they only
expose `@Composable` functions with callback parameters. The `:app:mobile` module
owns the navigation graph and wires everything together.

---

## Testing Navigation

```kotlin
@HiltAndroidTest
class NavigationTest {

    @get:Rule(order = 0)
    var hiltRule = HiltAndroidRule(this)

    @get:Rule(order = 1)
    val composeTestRule = createAndroidComposeRule<MainActivity>()

    @Test
    fun test1() {
        // Click item 0 → navigates to details
        composeTestRule.onNodeWithTag("item_0").performClick()

        // Verify back button exists on details screen
        composeTestRule.onNodeWithTag("nav_icon").assertExists()

        // Click back → returns to list
        composeTestRule.onNodeWithTag("nav_icon").performClick()
    }
}
```

This is a **cross-module E2E test** in `:test:navigation` that uses `com.android.test`
plugin to target `:app:mobile`. It tests the full navigation flow end-to-end.

---

## Dynamic Feature Module

`MainActivity` also logs installed dynamic feature modules:

```kotlin
val splitInstallManager = SplitInstallManagerFactory.create(context)
splitInstallManager.installedModules.addOnSuccessListener { moduleNames ->
    Timber.d("Installed modules: ${moduleNames.joinToString()}")
}
```

The `:dynamic` module (empty, install-time, fused) is referenced in `:app:mobile`'s
`build.gradle.kts` via `dynamicFeatures += setOf(":dynamic")`. The module has no
source code but demonstrates the dynamic-feature Gradle setup.
