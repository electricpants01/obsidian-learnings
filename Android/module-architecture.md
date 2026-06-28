# Multi-Module Architecture — Modularization Branch

## Why Modularize?

A single `:app` module works for small apps. As the app grows, a monolithic module causes:

- **Slow builds** — changing one line recompiles everything
- **Tight coupling** — classes accidentally depend on each other
- **Large PRs** — changes span an unbounded scope
- **Broken tests** — no clear ownership boundaries

Modularization enforces **separation of concerns** at the build system level.
If module A doesn't depend on module B, A's code cannot reference B's classes.

---

## Module Taxonomy (What Goes Where)

This project defines 6 module types, each with a specific role:

### 1. `:app:*` — Application Modules

**Purpose:** The executable entry point. Wires everything together.

**Contains:**
- `Application` class (`@HiltAndroidApp`)
- `Activity` class (`@AndroidEntryPoint`)
- Navigation graph (wires feature Routes together)
- App-level theme
- `AndroidManifest.xml`

**Does NOT contain:**
- Screens, ViewModels, business logic, data access

**In this project:** `:app:mobile`, `:app:wear`

**build.gradle.kts signature:**
```kotlin
plugins {
    alias(libs.plugins.sample.android.application)  // applies com.android.application
    alias(libs.plugins.sample.compose)               // enables Compose + BOM + test deps
}
dependencies {
    implementation(project(":feature:list"))
    implementation(project(":feature:details"))
    implementation(project(":core:ui"))
    // navigation, Compose, lifecycle libs
}
```

---

### 2. `:feature:*` — Feature Modules

**Purpose:** One screen or user-facing feature. Self-contained user journey.

**Contains:**
- `XxxRoute` composable (public entry point)
- `XxxViewModel` (`@HiltViewModel`)
- `XxxUiState` (`sealed interface`)
- UI-specific data classes
- Screen-internal composables (private to the module)
- Android instrumented tests

**Does NOT contain:**
- Navigation wiring (that's in `:app`)
- Knowledge of other features
- Data access implementation (depends on `:data` for that)

**In this project:** `:feature:list`, `:feature:details`, `:feature:wear:home`

**build.gradle.kts signature:**
```kotlin
plugins {
    alias(libs.plugins.sample.android.library)   // applies com.android.library
    alias(libs.plugins.sample.compose)           // enables Compose
}
dependencies {
    implementation(project(":core:ui"))
    implementation(project(":data"))
    // Compose, Material3, Hilt navigation
}
```

**Public API of a feature module:** The `Route` composable + its callback parameters.

```kotlin
// :feature:list exposes this:
@Composable
fun ListRoute(
    onGoToItem: (Long) -> Unit,   // ← navigation callback
    modifier: Modifier = Modifier,
    viewModel: ListViewModel = hiltViewModel()
)
```

Everything else is `internal` or `private`.

---

### 3. `:core:*` — Core/Shared Modules

**Purpose:** Reusable infrastructure used by multiple feature modules. No business logic.

**Types of core modules:**

| Module | Type | Contains |
|---|---|---|
| `:core:ui` | Android Library + Compose | Shared composables (`Loading()`) |
| `:core:util` | Pure Kotlin (no Android) | Utility functions (`timestampToReadableDate()`) |
| `:core:testing` | Android Library | Test runner, `HiltActivity`, test helpers |

**build.gradle.kts signatures:**

```kotlin
// :core:ui — Android + Compose
plugins {
    alias(libs.plugins.sample.android.library)
    alias(libs.plugins.sample.compose)
}
dependencies {
    implementation(libs.androidx.compose.ui)
    implementation(libs.androidx.compose.material3)
}

// :core:util — Pure Kotlin, no Android dependency at all
plugins {
    alias(libs.plugins.kotlin.jvm)   // kotlin-jvm, not android-library
}

// :core:testing — Android library, no Compose needed
plugins {
    alias(libs.plugins.sample.android.library)
}
dependencies {
    // Hilt testing, test runner deps
}
```

**Key insight:** `:core:util` uses `kotlin-jvm` plugin — it has zero Android dependencies.
This means it compiles fast on the JVM and can be tested without an emulator.

---

### 4. `:data` — Data Layer Module

**Purpose:** Data models, repository interfaces, repository implementations, DI modules.

**Contains:**
- Domain model (`MyModel`)
- Repository interface (`MyModelRepository`)
- Repository implementation (`DefaultMyModelRepository`)
- Hilt DI module (`DataModule`)
- JVM unit tests (fast, no emulator needed)

**Does NOT contain:**
- Android-specific code (no Activities, ViewModels, Composables)
- UI code

**In this project:** `:data`

**build.gradle.kts signature:**
```kotlin
plugins {
    alias(libs.plugins.sample.android.library)  // needs Android for coroutines-android
}
dependencies {
    implementation(libs.kotlinx.coroutines.android)
    testImplementation(libs.junit)
    testImplementation(libs.kotlinx.coroutines.test)
}
```

**Note:** No Compose plugin needed — the data layer never touches UI.

---

### 5. `:dynamic` — Dynamic Feature Module

**Purpose:** Code delivered on-demand via Google Play (not in the initial install).

**Contains:** (in this project) Nothing — it's an empty shell demonstrating the setup.

**Delivery modes:**
- `install-time` — downloaded during app install (the module is always present)
- `on-demand` — downloaded when the app requests it
- `conditional` — downloaded based on device conditions

**In this project:** `:dynamic` (install-time, fused=true)

**build.gradle.kts signature:**
```kotlin
plugins {
    alias(libs.plugins.sample.dynamic)   // applies com.android.dynamic-feature
}
dependencies {
    implementation(project(":app:mobile"))  // must depend on the base app
}
```

---

### 6. `:test:*` — Test-Only Modules

**Purpose:** Cross-module integration tests that span multiple features.

**Uses `com.android.test` plugin** — not `com.android.application` or library.
This creates a separate APK that tests the real app APK as a black box.

**In this project:** `:test:navigation`

**build.gradle.kts signature:**
```kotlin
plugins {
    alias(libs.plugins.sample.android.test)  // applies com.android.test
}
android {
    targetProjectPath = ":app:mobile"  // ← targets the app module
}
dependencies {
    implementation(project(":app:mobile"))
    implementation(project(":core:testing"))
    implementation(project(":data"))
    implementation(project(":feature:list"))
    implementation(project(":feature:details"))
}
```

---

## Dependency Rules

### Rule 1: Depend on interfaces, not implementations

Module dependencies should follow the **dependency inversion** pattern:

```
:feature:list ──depends-on──► :data (MyModelRepository interface)
                                  ↑
                                  implements
                                  │
                        DefaultMyModelRepository (in :data)
```

The feature module knows about the `interface`, not the implementation.
Hilt binds the implementation at runtime via `DataModule`.

### Rule 2: Features depend on data; data does NOT depend on features

```
✅ :feature:list → :data
✅ :feature:details → :data
❌ :data → :feature:list   (never)
```

### Rule 3: Features never depend on each other

```
❌ :feature:list → :feature:details
❌ :feature:details → :feature:list
```

Communication between features goes through:
- Navigation (app module wires Route callbacks)
- Shared data (both observe the same repository)
- Never direct class imports

### Rule 4: App depends on everything it needs

```
:app:mobile → :feature:list
:app:mobile → :feature:details
:app:mobile → :core:ui
```

### Rule 5: Core modules don't depend on features or data

```
✅ :core:ui → (nothing, just Compose/Material3)
✅ :core:util → (nothing, just Kotlin stdlib)
❌ :core:ui → :feature:list   (never)
```

---

## Convention Plugins — Eliminating build.gradle.kts Boilerplate

Without convention plugins, every module's `build.gradle.kts` would repeat:

```kotlin
android { compileSdk = 33; minSdk = 30; ... }
compileOptions { sourceCompatibility = JavaVersion.VERSION_17; ... }
dependencies {
    implementation(hilt.android)
    kapt(hilt.compiler)
    // ... 10+ repeated lines
}
```

With convention plugins, each module declares **only what's unique**:

```kotlin
// :feature:list — just 3-4 lines of unique config
plugins {
    alias(libs.plugins.sample.android.library)
    alias(libs.plugins.sample.compose)
}
android { namespace = "com.google.samples.modularization.feature.list" }
dependencies {
    implementation(project(":core:ui"))
    implementation(project(":data"))
}
```

### The 5 Convention Plugins

| Plugin ID | Applies | Used By |
|---|---|---|
| `sample.android.application` | `com.android.application` + Hilt + Kotlin | `:app:mobile`, `:app:wear` |
| `sample.android.library` | `com.android.library` + Hilt + Kotlin | `:feature:*`, `:core:ui`, `:core:testing`, `:data` |
| `sample.android.test` | `com.android.test` + Kotlin + Hilt + test deps | `:test:navigation` |
| `sample.compose` | Compose + BOM + test deps — pair with app or library | any module that uses Compose |
| `sample.dynamic` | `com.android.dynamic-feature` + Kotlin | `:dynamic` |

### How Convention Plugins Work

```
build-logic/                      ← included build (includeBuild("build-logic"))
  └── convention/
      ├── build.gradle.kts        ← registers the 5 plugins
      └── src/main/kotlin/
          ├── Android.kt           ← shared: compileSdk=33, minSdk=30, Java 17, HiltTestRunner
          ├── Compose.kt           ← shared: compose feature + Compose BOM + UI test deps
          ├── AndroidApplicationConventionPlugin.kt
          ├── AndroidLibraryConventionPlugin.kt
          ├── AndroidTestConventionPlugin.kt
          ├── ComposeConventionPlugin.kt
          └── DynamicFeatureConventionPlugin.kt
```

`build-logic` is an **included build** (declared in root `settings.gradle.kts`
via `includeBuild("build-logic")`). Its plugins are available to all modules
in the project without publishing to a repository.

### Plugin Registration

In `build-logic/convention/build.gradle.kts`:
```kotlin
gradlePlugin {
    plugins {
        register("sampleAndroidApplication") {
            id = "sample.android.application"
            implementationClass = "AndroidApplicationConventionPlugin"
        }
        // ... 4 more registrations
    }
}
```

In `gradle/libs.versions.toml`:
```toml
[plugins]
sample-android-application = { id = "sample.android.application", version = "unspecified" }
```

In a module's `build.gradle.kts`:
```kotlin
plugins {
    alias(libs.plugins.sample.android.application)
}
```

### What Each Convention Plugin Provides

**`sample.android.library`** (used by 6 modules):
```kotlin
apply plugin: "com.android.library"
apply plugin: "dagger.hilt.android.plugin"
apply plugin: "org.jetbrains.kotlin.android"
apply plugin: "org.jetbrains.kotlin.kapt"

android { compileSdk = 33; minSdk = 30; Java 17; testInstrumentationRunner = HiltTestRunner }

dependencies {
    hilt.android              // implementation
    hilt.compiler             // kapt
    hilt.android.testing      // androidTestImplementation
    hilt.android.compiler     // kaptAndroidTest
}
```

**`sample.compose`** (opted-in by 4 modules):
```kotlin
android { buildFeatures { compose = true } }
composeOptions { kotlinCompilerExtensionVersion = "1.4.3" }

dependencies {
    compose-bom (platform)
    compose-ui-test-junit4    // androidTestImplementation
    project(":core:testing")  // androidTestImplementation
}
```

---

## Version Catalog — Single Source of Truth

`gradle/libs.versions.toml` centralizes all dependency coordinates:

```toml
[versions]
kotlin = "1.8.10"
hilt = "2.45"
androidxComposeBom = "2023.03.00"
# ...

[libraries]
hilt-android = { module = "com.google.dagger:hilt-android", version.ref = "hilt" }
# ...

[plugins]
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
# ...
```

Benefits:
- Update a version in one place, all modules get it
- No version mismatch bugs between modules
- Easy to audit what's in use

---

## When to Create a New Module

### Create a new `:feature:*` module when:

- You're adding a new screen that doesn't belong to any existing feature
- The screen has its own ViewModel, UiState, and composables
- The screen requires different dependencies than existing features

### Create a new `:core:*` module when:

- You have code used by 3+ feature modules
- The shared code compiles independently of other modules
- Tests for the shared code can run without an emulator

### Create a new `:data` module when:

- You need a second data source (e.g., `:data:network` separate from `:data:local`)
- The data layer grows large enough to split by concern

### Do NOT create a new module when:

- You only have 1-2 classes — keep them in the module they belong to
- The code is only used by one feature — keep it in that feature module
- You're not sure — start in an existing module, extract later

### Signs you should split a module:

- The build time for that module is noticeably slow
- Two developers constantly merge-conflict in the same module
- You find yourself writing `internal` on everything because the module is too big
- A module depends on libraries that most of its code doesn't use

---

## Module Dependency Graph (Visual)

```
                    ┌─────────────────────┐
                    │   :app:mobile        │
                    │  (application)       │
                    └──┬───────┬──────┬────┘
                       │       │      │
              ┌────────┘       │      └──────────┐
              ▼                ▼                  ▼
    ┌──────────────┐  ┌──────────────┐   ┌──────────────┐
    │ :feature:list│  │:feature:     │   │  :core:ui    │
    │  (library)   │  │ details      │   │  (library)   │
    └───┬────┬─────┘  │  (library)   │   └──────────────┘
        │    │        └──────┬───────┘
        │    │               │
        │    │    ┌──────────┘
        │    │    │
        │    ▼    ▼
        │  ┌──────────────┐
        │  │    :data     │
        ▼  │  (library)   │
  ┌─────────┐             │
  │:core:   │             │
  │ util    │             │
  │(kotlin) │             │
  └─────────┘             │
                          │
  ┌───────────────────────┘
  │
  ▼
┌──────────────────┐
│   :dynamic       │
│ (dynamic-feature)│──depends-on──► :app:mobile
└──────────────────┘

  ┌──────────────────────┐
  │ :test:navigation     │
  │  (com.android.test)  │──targets──► :app:mobile
  └──────────────────────┘
```

Arrows point from dependent to dependency. `:app:mobile` depends on `:feature:list`.

---

## Real build.gradle.kts Comparison

### Most complex (`:app:mobile`)
```kotlin
plugins {
    alias(libs.plugins.sample.android.application)
    alias(libs.plugins.sample.compose)
}
android {
    namespace = "com.google.samples.modularization"
    defaultConfig { applicationId = "com.google.samples.modularization" }
    dynamicFeatures += setOf(":dynamic")
}
dependencies {
    implementation(project(":core:ui"))
    implementation(project(":feature:list"))
    implementation(project(":feature:details"))
    // + 8 library dependencies
}
```

### Simplest (`:core:util`)
```kotlin
plugins { alias(libs.plugins.kotlin.jvm) }
```

No Android, no dependencies — just `kotlin-jvm`. That's the entire file.

### Feature module (`:feature:list`)
```kotlin
plugins {
    alias(libs.plugins.sample.android.library)
    alias(libs.plugins.sample.compose)
}
android { namespace = "com.google.samples.modularization.feature.list" }
dependencies {
    implementation(project(":core:ui"))
    implementation(project(":core:util"))
    implementation(project(":data"))
    // + 3 library dependencies (compose-ui, material3, hilt-navigation-compose)
}
```
