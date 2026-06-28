# AI Documentation — Modularization Branch Reference

This folder contains detailed reference documentation for the key technologies
used in the **`multimodule`** branch of the **Android Architecture Samples**.
Each file distills the concrete patterns found in the actual source code, so an AI
(or a developer) can quickly understand what to look for and how things are wired together.

## Index

### Architecture & Framework

| File | Topic |
|---|---|
| [`module-architecture.md`](module-architecture.md) | Multi-module architecture — module types, dependency rules, convention plugins, when to split |
| [`compose.md`](compose.md) | Jetpack Compose — screens, state, `hiltViewModel()`, `collectAsState()` |
| [`flow.md`](flow.md) | Kotlin Flow + Coroutines — `StateFlow`, `stateIn`, `flatMapLatest`, `MutableStateFlow` |
| [`hilt.md`](hilt.md) | Hilt Dependency Injection — `@Binds`, `@HiltViewModel`, `SingletonComponent` |
| [`navigation-compose.md`](navigation-compose.md) | Navigation Compose — `NavHost`, `composable()`, `navArgument`, `SavedStateHandle` |
| [`testing.md`](testing.md) | Testing strategy — 3 tiers (JVM unit, Compose UI, E2E navigation), `HiltTestRunner`, `com.android.test` |

### Build (`build/`)

| File | Topic |
|---|---|
| [`build/convention-plugins.md`](build/convention-plugins.md) | Convention plugins — how `build-logic/` works, the 5 plugins, plugin registration, included builds |
| [`build/version-catalogs.md`](build/version-catalogs.md) | Version catalogs — `libs.versions.toml`, BOMs, `unspecified` versions, catalog sharing |
| [`build/build-optimization.md`](build/build-optimization.md) | Build optimization — caching, parallel, configuration cache, `nonTransitiveRClass`, `buildConfig` |

### Platforms (`platforms/`)

| File | Topic |
|---|---|
| [`platforms/wear-os.md`](platforms/wear-os.md) | Wear OS — `ScalingLazyColumn`, `Chip`, `SwipeDismissableNavController`, shared modules |
| [`platforms/dynamic-features.md`](platforms/dynamic-features.md) | Dynamic feature delivery — Play Feature Delivery, `install-time`, fusing, `SplitInstallManager` |

### Architecture (`architecture/`)

| File | Topic |
|---|---|
| [`architecture/kotlin-jvm-modules.md`](architecture/kotlin-jvm-modules.md) | JVM-only vs Android modules — `:core:util` pattern, when to use `kotlin-jvm` |

## Module Inventory (12 modules)

### App Modules
| Module | Purpose |
|---|---|
| `:app:mobile` | Phone app — `MyApplication` + `MainActivity` + `MainNavigation` |
| `:app:wear` | Wear OS app — standalone Wear Compose + `SwipeDismissableNavController` |

### Feature Modules (screens)
| Module | Purpose |
|---|---|
| `:feature:list` | Item list screen — `LazyColumn` + checkbox + click-to-navigate |
| `:feature:details` | Item detail screen — `Scaffold` + `CenterAlignedTopAppBar` + back nav |
| `:feature:wear:home` | Wear home screen — `ScalingLazyColumn` + `Chip` |

### Core Modules (shared)
| Module | Purpose |
|---|---|
| `:core:ui` | Shared `Loading()` composable — centered `CircularProgressIndicator` |
| `:core:util` | `timestampToReadableDate()` — pure Kotlin (no Android) |
| `:core:testing` | `HiltTestRunner` + `HiltActivity` — shared test infrastructure |
| `:core:lib` | Declared in `settings.gradle.kts` but **directory does not exist** |

### Data
| Module | Purpose |
|---|---|
| `:data` | `MyModel` + `MyModelRepository` + `DefaultMyModelRepository` + `DataModule` |

### Dynamic Feature
| Module | Purpose |
|---|---|
| `:dynamic` | Dynamic delivery module — empty shell, install-time, fused |

### Test
| Module | Purpose |
|---|---|
| `:test:navigation` | Cross-module navigation E2E test |

## Project Structure

```
├── app/
│   ├── mobile/src/main/java/.../
│   │   ├── MyApplication.kt          ← @HiltAndroidApp
│   │   └── ui/
│   │       ├── MainActivity.kt       ← @AndroidEntryPoint, hosts Compose
│   │       ├── MyAppTheme.kt         ← Material3 theme
│   │       └── Navigation.kt         ← NavHost + composable destinations
│   └── wear/src/main/java/.../
│       ├── MyApplication.kt
│       └── ui/Navigation.kt
│
├── feature/
│   ├── list/src/main/java/.../feature/list/ui/
│   │   ├── ListRoute.kt              ← @Composable screen
│   │   ├── ListViewModel.kt          ← @HiltViewModel
│   │   ├── ListUiState.kt            ← sealed interface (Loading / Success)
│   │   └── MyModelUiState.kt         ← UI data class
│   ├── details/src/main/java/.../feature/details/ui/
│   │   ├── DetailsRoute.kt
│   │   ├── DetailsViewModel.kt       ← SavedStateHandle extracts nav arg
│   │   └── DetailsUiState.kt
│   └── wear/home/src/main/java/.../feature/wear/home/ui/
│       ├── HomeRoute.kt
│       ├── HomeViewModel.kt
│       └── HomeUiState.kt
│
├── core/
│   ├── ui/src/main/java/.../ui/
│   │   └── Loading.kt                ← @Composable Loading()
│   ├── util/src/main/java/.../util/
│   │   └── TimestampToDataString.kt  ← timestampToReadableDate()
│   └── testing/src/main/java/.../testing/
│       ├── HiltTestRunner.kt
│       └── HiltActivity.kt
│
├── data/
│   └── src/main/java/.../core/
│       ├── data/
│       │   ├── MyModel.kt            ← data class (id, title, description, timestamp, isBookmarked)
│       │   ├── MyModelRepository.kt  ← interface (observeAllModels, observeModelById, bookmark)
│       │   └── DefaultModelRepository.kt  ← in-memory MutableStateFlow impl
│       └── di/
│           └── DataModule.kt         ← @Binds DefaultMyModelRepository → MyModelRepository
│
├── test/navigation/src/main/java/.../test/navigation/
│   └── NavigationTest.kt             ← E2E: list → detail → back
│
└── build-logic/convention/src/main/kotlin/
    ├── Android.kt                     ← shared: compileSdk=33, minSdk=30, Java 17
    ├── Compose.kt                     ← shared: compose feature + BOM
    └── *.kt                           ← 5 convention plugins
```

## Architecture Overview (Call Hierarchy)

```
MainNavigation (NavHost)
  ├── "list" → ListRoute(@Composable)
  │     └── ListViewModel (@HiltViewModel)
  │           └── MyModelRepository.observeAllModels : Flow<List<MyModel>>
  │                 └── DefaultMyModelRepository (in-memory MutableStateFlow, 3 seed items)
  │
  └── "details/{id}" → DetailsRoute(@Composable)
        └── DetailsViewModel (@HiltViewModel)
              ├── SavedStateHandle.getStateFlow<Long?>("id", null) ← extracts nav arg
              └── MyModelRepository.observeModelById(id) : Flow<MyModel>
```

Every feature follows the same pattern:
1. Compose screen calls `hiltViewModel()` to get the ViewModel
2. ViewModel injects `MyModelRepository`, maps Flow → `StateFlow<UiState>`
3. UiState is a `sealed interface` with `Loading` and `Success(data)` variants
4. Screen uses `collectAsState()` and `when(state)` to render

## Dependency Graph

```
:app:mobile ──────────┐
  ├── :core:ui         │
  ├── :feature:list ───┤──── :data ─── DataModule
  │    └── :core:util  │
  ├── :feature:details─┘
  └── :dynamic (empty)

:app:wear ── :feature:wear:home ── :data ─┘

:test:navigation (targets :app:mobile)
  ├── :app:mobile
  ├── :core:testing
  ├── :data
  ├── :feature:list
  └── :feature:details
```

## Build System

- **Convention plugins** (`build-logic/`): 5 custom Gradle plugins centralize Android, Hilt, Compose, and test configuration
- **Version catalog** (`gradle/libs.versions.toml`): Kotlin 1.8.10, AGP 7.4.2, Hilt 2.45, Compose BOM 2023.03.00
- **Root project name**: `modularization`
- **Target**: `compileSdk=33`, `minSdk=30`, Java 17
