# Wear OS — Modularization Branch

## Overview

This project includes a **standalone Wear OS app** (`:app:wear`) that shares modules with the mobile app (`:app:mobile`). It demonstrates how to build for two form factors from one codebase.

---

## Module Structure

```
:app:wear                                 ← Wear app entry point
  ├── depends on :feature:wear:home       ← Wear screen
  └── depends on :data                    ← shared data layer

:feature:wear:home                        ← Wear feature module
  ├── depends on :data
  └── depends on :core:ui                 ← shared Loading() composable
```

Both mobile and Wear share `:data`, `:core:ui`, and `:core:util`. Each has its own app module, navigation, and feature screens.

---

## Wear vs Mobile: Side by Side

| Aspect | Mobile | Wear |
|---|---|---|
| **App module** | `:app:mobile` | `:app:wear` |
| **Plugin** | `sample.android.application` | `sample.android.application` |
| **Entry point** | `MainActivity` → `MyAppTheme { Surface { MainNavigation() } }` | `MainActivity` → `MaterialTheme { MainNavigation() }` |
| **Theme** | Material 3 `MyAppTheme` (light/dark) | `androidx.wear.compose.material.MaterialTheme` |
| **NavController** | `rememberNavController()` | `rememberSwipeDismissableNavController()` |
| **NavHost** | `NavHost` | `SwipeDismissableNavHost` |
| **Destinations** | 2 (`list`, `details/{id}`) | 1 (`home`) |
| **List widget** | `LazyColumn` + `ListItem` | `ScalingLazyColumn` + `Chip` |
| **Navigation** | Tap to navigate, back button | Swipe right to dismiss |

---

## Wear Navigation

```kotlin
@Composable
fun MainNavigation(
    navController: NavHostController = rememberSwipeDismissableNavController()
) {
    SwipeDismissableNavHost(navController = navController, startDestination = "home") {
        composable("home") {
            HomeRoute()
        }
    }
}
```

### `SwipeDismissableNavController`

- Replaces `NavController` on Wear OS
- Enables **swipe-to-dismiss** (swipe left-to-right to navigate back)
- Based on the physical gesture Wear users expect

### `SwipeDismissableNavHost`

- Wear-specific `NavHost` that handles swipe gesture detection
- Only one destination in this project — a real Wear app might have more

---

## Wear Composable: `ScalingLazyColumn` + `Chip`

```kotlin
// HomeRoute.kt (conceptual — actual uses HomeScreen → Content pattern)
@Composable
fun HomeRoute(
    viewModel: HomeViewModel = hiltViewModel(),
    modifier: Modifier = Modifier
) {
    val state by viewModel.uiState.collectAsState()
    when (state) {
        HomeUiState.Loading -> Loading(modifier)
        is HomeUiState.Success -> {
            ScalingLazyColumn(modifier = modifier.fillMaxSize()) {
                items(state.data) { model ->
                    item {
                        Chip(
                            onClick = { /* action */ },
                            label = { Text(model.title) }
                        )
                    }
                }
            }
        }
    }
}
```

### Key Wear Composables

| Composable | Purpose | Mobile Equivalent |
|---|---|---|
| `ScalingLazyColumn` | Scrollable list optimized for round screens — items scale as they approach the center | `LazyColumn` |
| `Chip` | Tappable card — compact, touch-friendly for small screens | `ListItem` + `Card` |
| `MaterialTheme` (wear) | Wear-specific color scheme and typography | `MaterialTheme` (Material 3) |

### `ScalingLazyColumn` — How it differs from `LazyColumn`

On a round watch face, the center of the screen has more horizontal space than the top/bottom. `ScalingLazyColumn` **scales items up** as they scroll into the center and **scales them down** as they approach the edges — making content readable at every position.

---

## Wear Dependencies

```kotlin
// :app:wear/build.gradle.kts (conceptual — actual uses convention plugins)
dependencies {
    implementation("androidx.wear.compose:compose-foundation:1.0.0")
    implementation("androidx.wear.compose:compose-material:1.0.0")
    implementation("androidx.wear.compose:compose-navigation:1.0.0")
}
```

In the version catalog:

```toml
[versions]
composeWear = "1.0.0"

[libraries]
androidx-wear-compose-foundation = { group = "androidx.wear.compose", name = "compose-foundation", version.ref = "composeWear" }
androidx-wear-compose-material = { group = "androidx.wear.compose", name = "compose-material", version.ref = "composeWear" }
androidx-wear-compose-navigation = { group = "androidx.wear.compose", name = "compose-navigation", version = "1.0.0" }
```

---

## Shared Modules Between Mobile and Wear

Both apps depend on the same modules without any Wear-specific variants:

```kotlin
// :app:mobile/build.gradle.kts
implementation(project(":data"))           // shared
implementation(project(":core:ui"))        // shared

// :app:wear/build.gradle.kts
implementation(project(":data"))           // same module!
implementation(project(":core:ui"))        // same module!
```

This works because:
- `:data` is a pure data layer (no UI code, no form-factor assumptions)
- `:core:ui` provides `Loading()` — a generic composable that works on any form factor
- `:core:util` provides `timestampToReadableDate()` — pure Kotlin, no Android

**If you needed form-factor-specific shared code**, you'd split `:core:ui` into:
- `:core:ui` — shared composables
- `:core:ui-wear` — Wear-specific composables (depends on wear-compose)

---

## Wear Theme

```kotlin
// :app:wear
setContent {
    MaterialTheme {         // ← androidx.wear.compose.material.MaterialTheme
        MainNavigation()
    }
}
```

Wear `MaterialTheme` is a separate dependency from mobile `MaterialTheme`. It provides:
- Smaller default text sizes optimized for watch faces
- High-contrast color schemes
- Circular screen-aware layouts

---

## When to Build a Wear Module

| Scenario | Approach |
|---|---|
| Simple notification forwarding | Use `NotificationListenerService` — no Wear app needed |
| Custom watch face | Wear standalone app |
| Companion experience (control phone app from watch) | Wear + mobile sharing `:data` |
| Separate data sources per form factor | Wear-only `:data:wear` module |
| Same data, different UI | Shared `:data`, separate `:feature:wear:*` modules |
