# Testing Strategy — Modularization Branch

## Overview

This project has **3 tiers of tests**, each in a different module type:

| Tier | Module | Type | Plugin | Runs On | Example |
|---|---|---|---|---|---|
| **Unit** | `:data` | JVM | `sample.android.library` | Host JVM (no emulator) | `DefaultMyModelRepositoryTest` |
| **Feature UI** | `:feature:list` | Instrumented | `sample.android.library` | Android emulator/device | `ListFeatureTest` |
| **E2E Navigation** | `:test:navigation` | Instrumented | `sample.android.test` | Android emulator/device | `NavigationTest` |

---

## Tier 1: Unit Tests (JVM)

**Location:** `data/src/test/java/.../data/`

```kotlin
class DefaultMyModelRepositoryTest {

    @OptIn(ExperimentalCoroutinesApi::class)
    @Test
    fun `ensure item is bookmarked`() = runTest {
        val repository = DefaultMyModelRepository()
        val items = repository.observeAllModels.first()
        assert(items.none { it.isBookmarked })
        val firstItemId = items.first().id
        repository.bookmark(firstItemId, true)
        val firstItem = repository.observeModelById(firstItemId).first()
        assert(firstItem.isBookmarked)
    }
}
```

### Key Patterns

| Pattern | Why |
|---|---|
| **Direct instantiation** `DefaultMyModelRepository()` | No DI needed — the class has no external dependencies |
| **`runTest {}`** from `kotlinx-coroutines-test` | Virtual time test scope — `suspend` calls run synchronously |
| **`Flow.first()`** | Suspends until the flow emits one value, then returns it |
| **No Android dependency** | Runs on host JVM via Gradle `test` task — ultra-fast |

### Dependencies
```kotlin
// data/build.gradle.kts
testImplementation(libs.junit)
testImplementation(libs.kotlinx.coroutines.test)
```

### When to use JVM tests
- Repository logic (CRUD, filtering, mapping)
- Utility functions (`:core:util`)
- Anything that doesn't touch Android framework classes

---

## Tier 2: Feature UI Tests (Instrumented)

**Location:** `feature/list/src/androidTest/java/.../feature/list/`

```kotlin
@HiltAndroidTest
class ListFeatureTest {

    @get:Rule(order = 0)
    var hiltRule = HiltAndroidRule(this)

    @get:Rule(order = 1)
    val composeTestRule = createAndroidComposeRule<HiltActivity>()

    @Test
    fun `test item is displayed`() {
        composeTestRule.setContent {
            ListRoute(onGoToItem = { /* no-op */ })
        }
        composeTestRule.onNodeWithTag("item_1").assertExists()
    }
}
```

### Key Patterns

| Pattern | Why |
|---|---|
| **`@HiltAndroidTest`** | Enables Hilt injection in instrumented tests |
| **`HiltAndroidRule`** | Manages Hilt component lifecycle for the test |
| **`order = 0`** | Hilt must initialize before Activity rule (order = 1) |
| **`createAndroidComposeRule<HiltActivity>()`** | Uses lightweight `HiltActivity`, not real `MainActivity` |
| **`setContent { ListRoute(...) }`** | Renders the feature in isolation — no nav graph, no other screens |
| **`onNodeWithTag("item_1").assertExists()`** | Finds composable by `testTag(...)` modifier |
| **No-op callbacks** | `onGoToItem = {}` — tests don't care about navigation |

### Why `HiltActivity` instead of `MainActivity`?

`MainActivity` depends on all feature modules + full navigation graph. `HiltActivity` is just:

```kotlin
@AndroidEntryPoint
class HiltActivity : ComponentActivity()
```

Benefits:
- **Faster startup** — no nav graph, no theme, no other features
- **True isolation** — tests the feature, not the app
- **No circular deps** — `:feature:list` doesn't need to depend on `:app:mobile`

### The `testTag` Contract

Feature modules declare tags that become public API:

```kotlin
// :feature:list
ListItem(modifier = Modifier.testTag("item_$index"))

// :feature:details
IconButton(onGoBack, modifier = Modifier.testTag("nav_icon"))
```

Other test modules (`:test:navigation`) depend on these tags existing.

---

## Tier 3: Cross-Module E2E Tests (Instrumented)

**Location:** `test/navigation/src/main/java/.../test/navigation/`

```kotlin
@HiltAndroidTest
class NavigationTest {

    @get:Rule(order = 0)
    var hiltRule = HiltAndroidRule(this)

    @get:Rule(order = 1)
    val composeTestRule = createAndroidComposeRule<MainActivity>()

    @Test
    fun test1() {
        composeTestRule.onNodeWithTag("item_0").performClick()  // navigate
        composeTestRule.onNodeWithTag("nav_icon").assertExists() // verify details screen
        composeTestRule.onNodeWithTag("nav_icon").performClick() // go back
    }
}
```

### Key Differences from Feature Tests

| Aspect | Feature Test | E2E Test |
|---|---|---|
| Host Activity | `HiltActivity` | `MainActivity` |
| Scope | Single feature in isolation | Full app with real navigation |
| Module plugin | `sample.android.library` | `sample.android.test` |
| Dependencies | `:data`, `:core:ui`, `:core:util` | `:app:mobile` + all features |
| What fails if broken | One feature | Integration wiring |

### How `com.android.test` Works

```
:test:navigation (com.android.test plugin → produces test APK)
  │
  │  depends on :app:mobile + :feature:* + :data
  │  targets :app:mobile (targetProjectPath)
  │
  ▼
  1. Builds :app:mobile APK (app under test)
  2. Builds :test:navigation APK (test APK)
  3. Installs both on device
  4. Runs tests in test APK against app APK
```

### build.gradle.kts
```kotlin
plugins { alias(libs.plugins.sample.android.test) }
android {
    namespace = "com.google.samples.modularization.test.navigation"
    targetProjectPath = ":app:mobile"        // ← which app to test
}
dependencies {
    implementation(project(":app:mobile"))    // needs the real app
    implementation(project(":core:testing"))   // HiltTestRunner, HiltActivity
    implementation(project(":data"))
    implementation(project(":feature:list"))
    implementation(project(":feature:details"))
}
```

---

## Test Infrastructure (`:core:testing`)

### `HiltTestRunner`

```kotlin
class HiltTestRunner : AndroidJUnitRunner() {
    override fun newApplication(cl: ClassLoader?, name: String?, context: Context?): Application {
        return super.newApplication(cl, HiltTestApplication::class.java.name, context)
    }
}
```

- Replaces app's `Application` with `HiltTestApplication`
- `HiltTestApplication` supports `@HiltAndroidTest` + allows overriding bindings
- Set globally via convention plugin: `testInstrumentationRunner = "...HiltTestRunner"`
- **Every instrumented test in the project uses this runner automatically**

### `HiltActivity`

```kotlin
@AndroidEntryPoint
class HiltActivity : ComponentActivity()
```

Minimal `@AndroidEntryPoint` activity — just enough for Compose to render inside.

---

## Rule Ordering: Why `order = 0` / `order = 1`

```kotlin
@get:Rule(order = 0)
var hiltRule = HiltAndroidRule(this)

@get:Rule(order = 1)
val composeTestRule = createAndroidComposeRule<HiltActivity>()
```

JUnit runs `@Rule` methods in ascending `order` value. If Hilt hasn't initialized before the Activity is created, the Activity can't inject dependencies. `order = 0` guarantees Hilt is ready first.

---

## What's NOT Tested (Intentional Gaps)

| Gap | Why |
|---|---|
| No ViewModel unit tests | ViewModels are thin wrappers around `map`/`stateIn`. Repository tests cover the logic. |
| No fake Hilt modules | `DefaultMyModelRepository` has no external deps, so nothing to fake. In a real app, you'd use `@TestInstallIn` to swap Room/Retrofit with fakes. |
| No snapshot/screenshot tests | Out of scope for this sample. |

---

## Test Decision Matrix

| What to test | Module | Plugin | Host | Pattern |
|---|---|---|---|---|
| Repository logic, utilities | `:data/src/test` | JUnit + coroutines-test | JVM | `runTest {}` |
| Isolated feature UI | `:feature:*/src/androidTest` | `sample.android.library` | Emulator | `createAndroidComposeRule<HiltActivity>()` |
| Navigation across features | `:test:navigation` | `sample.android.test` | Emulator | `createAndroidComposeRule<MainActivity>()` |

---

## Running Tests

```bash
./gradlew test                                    # all unit tests (JVM, fast)
./gradlew connectedAndroidTest                    # all instrumented tests (needs emulator)
./gradlew :data:test                              # specific module unit tests
./gradlew :feature:list:connectedAndroidTest      # specific feature UI tests
./gradlew :test:navigation:connectedAndroidTest   # E2E navigation tests
```
