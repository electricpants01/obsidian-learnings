# Hilt Dependency Injection — Patterns in the Modularization Branch

## Key Takeaways

1. **`@HiltAndroidApp`** on `MyApplication` — triggers Hilt code generation.
2. **`@HiltViewModel` + `@Inject constructor`** on every ViewModel — Hilt resolves all deps.
3. **`@Module @InstallIn(SingletonComponent::class)`** — single module using `@Binds`.
4. **No qualifiers, no `@Provides`** — this branch is minimal.
5. **`hiltViewModel()`** in Compose to obtain the ViewModel instance.
6. **Test overrides** via `HiltTestRunner` + `HiltTestApplication`.

---

## Files to Check

| File | Module | Purpose |
|---|---|---|
| `MyApplication.kt` | `:app:mobile` | `@HiltAndroidApp` entry point |
| `DataModule.kt` | `:data` | `@Binds` `DefaultMyModelRepository` → `MyModelRepository` |
| `ListViewModel.kt` | `:feature:list` | `@HiltViewModel` + `@Inject constructor` |
| `DetailsViewModel.kt` | `:feature:details` | `@HiltViewModel` + `@Inject constructor` + `SavedStateHandle` |
| `HomeViewModel.kt` | `:feature:wear:home` | `@HiltViewModel` + `@Inject constructor` |
| `DefaultMyModelRepository.kt` | `:data` | `@Inject constructor` — eligible for Hilt injection |
| `HiltTestRunner.kt` | `:core:testing` | Custom test runner using `HiltTestApplication` |
| `HiltActivity.kt` | `:core:testing` | Empty `@AndroidEntryPoint` activity for Compose UI tests |

---

## `@HiltAndroidApp` Entry Point

```kotlin
@HiltAndroidApp
class MyApplication : Application()
```

This triggers Hilt's code generation for the **SingletonComponent**. Every Hilt-based
Android app must have exactly one `@HiltAndroidApp`-annotated `Application` subclass.

Both `:app:mobile` and `:app:wear` have their own `MyApplication` with this annotation.

---

## ViewModel Injection

```kotlin
@HiltViewModel
class ListViewModel @Inject constructor(
    private val myModelRepository: MyModelRepository
) : ViewModel()
```

Two annotations work together:
- `@HiltViewModel` — tells Hilt to generate a `ViewModelProvider.Factory` for this ViewModel
- `@Inject constructor` — tells Hilt which constructor to use and what dependencies to provide

### `SavedStateHandle` — automatically available

```kotlin
@HiltViewModel
class DetailsViewModel @Inject constructor(
    private val myModelRepository: MyModelRepository,
    savedStateHandle: SavedStateHandle   // ← no module needed
) : ViewModel()
```

`SavedStateHandle` is always available to `@HiltViewModel` classes without any
explicit module registration. Hilt automatically provides it for ViewModels.
It's used here to read the `id` navigation argument (`savedStateHandle.getStateFlow<Long?>("id", null)`).

---

## Module: `@Binds` for Interface → Implementation

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class DataModule {

    @Binds
    abstract fun bindMyModelRepository(
        repository: DefaultMyModelRepository
    ): MyModelRepository
}
```

### How it works:

| Element | Meaning |
|---|---|
| `@Module` | This class contributes to the Hilt dependency graph |
| `@InstallIn(SingletonComponent::class)` | These bindings live in the application-scoped component |
| `abstract class` | Required for `@Binds` modules (Hilt generates the implementation) |
| `@Binds` | "When someone asks for `MyModelRepository`, give them `DefaultMyModelRepository`" |
| No `@Singleton` on the `@Binds` | The scope is inherited from the implementation class — see below |

### Where's `@Singleton`?

`DefaultMyModelRepository` uses plain `@Inject constructor()` without `@Singleton`.
In Hilt, when using `@Binds`, the scope annotation should be on the **implementation class**.
This branch demonstrates that omitting `@Singleton` means a new instance is created
for each injection (though with `SingletonComponent` and only one injection point,
the behavior is effectively singleton-like in practice).

---

## How Hilt Resolves the Graph

```
@HiltAndroidApp
  └── SingletonComponent
        └── DataModule: MyModelRepository → DefaultMyModelRepository

  └── ViewModelComponent
        ├── ListViewModel(MyModelRepository)
        ├── DetailsViewModel(MyModelRepository, SavedStateHandle)
        └── HomeViewModel(MyModelRepository)
```

`MyModelRepository` is bound in `SingletonComponent`, so it's available to all
ViewModels without explicit per-ViewModel configuration.

---

## Obtaining ViewModels in Compose

```kotlin
@Composable
fun ListRoute(
    onGoToItem: (Long) -> Unit,
    modifier: Modifier = Modifier,
    viewModel: ListViewModel = hiltViewModel()   // ← Hilt provides the ViewModel
) {
    val state by viewModel.uiState.collectAsState()
    // ...
}
```

`hiltViewModel()` from `androidx.hilt.navigation.compose` knows how to:
1. Find the nearest `@AndroidEntryPoint` activity (or `NavBackStackEntry`)
2. Use the generated `ViewModelProvider.Factory` to create the ViewModel
3. Return the ViewModel scoped to the current navigation destination

**Why as a default argument?** It allows tests to pass a fake ViewModel, and allows
previews to provide mock data.

---

## `@AndroidEntryPoint` on Activities

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity()
```

This tells Hilt that `MainActivity` is an injection target. It doesn't inject
anything into the Activity directly — it enables child fragments/composables
to use `hiltViewModel()`.

---

## Convention Plugin Hilt Setup

Hilt is configured centrally in the build-logic convention plugins:

```kotlin
// AndroidLibraryConventionPlugin.kt (simplified)
plugins {
    id("dagger.hilt.android.plugin")
    id("kotlin-kapt")
}

dependencies {
    implementation(libs.hilt.android)
    kapt(libs.hilt.compiler)
    implementation(libs.hilt.navigation.compose)
    
    kaptTest(libs.hilt.compiler)
    testImplementation(libs.hilt.android.testing)
    
    kaptAndroidTest(libs.hilt.compiler)
    androidTestImplementation(libs.hilt.android.testing)
}
```

Every Android module that applies `sample.android.library` or `sample.android.application`
gets Hilt automatically — no per-module Hilt boilerplate.

---

## Testing with Hilt

### Test Runner

```kotlin
// core/testing/.../HiltTestRunner.kt
class HiltTestRunner : AndroidJUnitRunner() {
    override fun newApplication(...): Application =
        CustomTestRunner_Application.newApplication(HiltTestApplication::class.java, ...)
}
```

Set globally via the convention plugin:
```kotlin
testInstrumentationRunner = "com.google.samples.modularization.testing.HiltTestRunner"
```

### `HiltActivity` — Empty Test Host

```kotlin
@AndroidEntryPoint
class HiltActivity : ComponentActivity()
```

An empty activity used as a lightweight host for Compose UI tests. Tests use
`createAndroidComposeRule<HiltActivity>()` instead of launching the real `MainActivity`.

### Feature Test Pattern

```kotlin
@HiltAndroidTest
class ListFeatureTest {

    @get:Rule(order = 0)
    var hiltRule = HiltAndroidRule(this)       // ← Hilt injection rule

    @get:Rule(order = 1)
    val composeTestRule = createAndroidComposeRule<HiltActivity>()  // ← Compose test rule

    @Test
    fun `test item is displayed`() {
        composeTestRule.setContent {
            ListRoute(onGoToItem = { /* no-op */ })  // ← renders the feature in isolation
        }
        composeTestRule.onNodeWithTag("item_1").assertExists()
    }
}
```

**Order matters:** `HiltAndroidRule` must run first (order = 0) so Hilt is initialized
before the Activity is created (order = 1).

### Unit Test — No Hilt

```kotlin
class DefaultMyModelRepositoryTest {
    @Test
    fun `ensure item is bookmarked`() = runTest {
        val repository = DefaultMyModelRepository()   // ← direct instantiation
        // ... test bookmark logic
    }
}
```

The repository unit test doesn't use Hilt at all — it directly instantiates
`DefaultMyModelRepository()`. This is possible because the class has no
external dependencies.
