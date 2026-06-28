# Testing

For the full testing best practices guide, see [Collectibles Architecture & Best Practices](../docs/CollectiblesArchitectureAndBestPractices.md).

---

## Unit Tests

### Structure

Tests mirror the source structure:

```
src/test/java/.../
├── viewmodel/    # ViewModel tests
├── usecase/      # Use case tests
├── data/         # Repository / data source tests
└── api/          # API contract tests
```

### ViewModel Testing

Using `CollectiblesScaffold` test support:

```kotlin
@Test
fun `verify initial state`() = runTest {
    val viewModel = MyViewModel(...)
    val state = viewModel.uiState.first()
    assertTrue(state.isLoading)
}

@Test
fun `verify event handling`() = runTest {
    val viewModel = MyViewModel(...)
    viewModel.handleEvent(MyEvent.Refresh)
    val state = viewModel.uiState.last { !it.isLoading }
    assertEquals(expectedData, state.data)
}
```

### Flow Testing with Turbine

```kotlin
@Test
fun `verify effect emission`() = runTest {
    val viewModel = MyViewModel(...)
    viewModel.effects.test {
        viewModel.handleEvent(MyEvent.Submit)
        assertEquals(MyEffect.ShowSuccess, awaitItem())
    }
}
```

### Fakes over Mocks

Prefer fake implementations over mocked interfaces:

```kotlin
class FakeMyRepository : MyRepository {
    private val data = MutableStateFlow<AsyncState<List<Item>>>(AsyncState.Loading)

    override fun getItems(): Flow<AsyncState<List<Item>>> = data

    fun emitItems(items: List<Item>) {
        data.value = AsyncState.Success(items)
    }
}
```

---

## UI Tests

### Page Object Model

Each screen has a corresponding Page Object in `digitalCollectionsTestSupport`:

```kotlin
class MyScreenPageObject(composeTestRule: ComposeTestRule) {
    private val title = composeTestRule.onNodeWithText("My Screen")

    fun assertDisplayed() {
        title.assertIsDisplayed()
    }
}
```

### Compose Test Rules

```kotlin
@get:Rule
val composeTestRule = createComposeRule()

@Test
fun screen_displays_loading_then_content() {
    composeTestRule.setContent {
        MyTheme { MyScreen(viewModel = viewModel) }
    }
    composeTestRule.onNodeWithText("Loading").assertIsDisplayed()
}
```

---

## Test Support Modules

| Module | Contents |
|---|---|
| `digitalCollectionsTestSupport` | Shared POMs, Dagger test modules, paging test support |
| `digitalCollectionsInternalTestHelper` | Fakes, stubs, test rules, Espresso idling resources |

### Dagger Test Modules

Test modules override production bindings with fakes:

```kotlin
@TestModule
object TestMyFeatureModule {
    @Provides fun provideMyRepository(): MyRepository = FakeMyRepository()
}
```

See the [Internal Test Helper README](../digitalCollectionsInternalTestHelper/README.md) for the full test module reference.
