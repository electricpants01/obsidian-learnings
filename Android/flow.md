# Kotlin Flow + Coroutines — Patterns in the Modularization Branch

## Key Takeaways

1. **`StateFlow`** is the backbone — ViewModels expose `StateFlow<UiState>`.
2. **`MutableStateFlow`** in the repository — in-memory data with `getAndUpdate` for atomic mutations.
3. **`map`** transforms `MyModel` → `MyModelUiState` (formatting dates via `:core:util`).
4. **`stateIn()` with `SharingStarted.WhileSubscribed(5000)`** for lifecycle-aware collection.
5. **`flatMapLatest`** in DetailsViewModel to reactively switch on the nav argument ID.
6. **`filterNotNull`** to skip null states until the navigation argument is available.
7. **`viewModelScope.launch`** for one-shot operations (bookmark toggle).

---

## Data Flow: Repository → ViewModel → Compose

```
MutableStateFlow<List<MyModel>> (in-memory, 3 seed items)
  │
  ▼
MyModelRepository.observeAllModels : Flow<List<MyModel>>
  │
  ▼
ListViewModel
  │  .map { items → items.map { MyModelUiState } → Success }
  │  .stateIn(viewModelScope, WhileSubscribed(5000), Loading)
  │
  ▼
val uiState: StateFlow<ListUiState>
  │
  ▼
ListRoute
  │  val state by viewModel.uiState.collectAsState()
  │  when (state) { Loading → Loading(); is Success → Content(...) }
```

---

## Repository: `MutableStateFlow` + `getAndUpdate`

```kotlin
class DefaultMyModelRepository @Inject constructor() : MyModelRepository {

    private val allItems: MutableStateFlow<List<MyModel>> = MutableStateFlow(
        listOf(
            MyModel(id = 1, title = "Item 1", description = "...", timestamp = 1672368617954, isBookmarked = false),
            MyModel(id = 2, title = "Item 2", description = "...", timestamp = 1664678230741, isBookmarked = false),
            MyModel(id = 3, title = "Item 3", description = "...", timestamp = 1667884312189, isBookmarked = false)
        )
    )

    override val observeAllModels: Flow<List<MyModel>> = allItems   // ← exposed as read-only Flow

    override fun observeModelById(id: Long): Flow<MyModel> =
        observeAllModels.map { items ->
            items.firstOrNull { it.id == id }
                ?: throw NoSuchElementException("$id not found")
        }

    override suspend fun bookmark(id: Long, isBookmarked: Boolean) {
        allItems.getAndUpdate { items ->             // ← atomic read-modify-write
            items.map { model ->
                if (model.id == id) model.copy(isBookmarked = isBookmarked)
                else model
            }
        }
    }
}
```

### Why `getAndUpdate`?

`getAndUpdate` atomically reads the current value, applies the transformation, and
writes it back. It guarantees no lost updates if two coroutines call `bookmark()`
concurrently. If you used `allItems.value = allItems.value.map { }` instead, concurrent
writes could stomp each other.

### Why expose `Flow` not `MutableStateFlow`?

The repository interface exposes `Flow<List<MyModel>>`, not `MutableStateFlow`. This
prevents consumers from mutating the state directly — only the repository controls
writes through `bookmark()`.

---

## ListViewModel: `map` + `stateIn`

```kotlin
@HiltViewModel
class ListViewModel @Inject constructor(
    private val myModelRepository: MyModelRepository
) : ViewModel() {

    val uiState: StateFlow<ListUiState> = myModelRepository
        .observeAllModels
        .map<List<MyModel>, ListUiState> { items ->
            items.map { myModel ->
                MyModelUiState(
                    id = myModel.id,
                    title = myModel.title,
                    date = timestampToReadableDate(myModel.timestamp),
                    isBookmarked = myModel.isBookmarked
                )
            }.let(::Success)
        }
        .stateIn(
            viewModelScope,
            SharingStarted.WhileSubscribed(5000),
            ListUiState.Loading
        )

    fun bookmark(id: Long, isBookmarked: Boolean) {
        viewModelScope.launch {
            myModelRepository.bookmark(id, isBookmarked)
        }
    }
}
```

### Step by step:

1. `observeAllModels` emits `List<MyModel>` whenever data changes
2. `.map { }` transforms each `MyModel` → `MyModelUiState` (date formatting via `:core:util`), wraps in `Success`
3. `.stateIn(viewModelScope, WhileSubscribed(5000), Loading)` converts the cold Flow into a hot `StateFlow`
   - `viewModelScope` — the coroutine scope (auto-cancelled when ViewModel is cleared)
   - `WhileSubscribed(5000)` — stops collecting 5 seconds after the last subscriber disappears
   - `Loading` — the initial value before the first emission

### `WhileSubscribed(5000)` — Why?

When the composable goes off-screen, `stateIn` stops collecting after 5 seconds,
preventing unnecessary work. When the composable comes back, it resubscribes.
The 5-second delay avoids stopping/restarting on brief configuration changes
(e.g., screen rotation).

---

## DetailsViewModel: `flatMapLatest` + `filterNotNull` + `SavedStateHandle`

```kotlin
@HiltViewModel
class DetailsViewModel @Inject constructor(
    private val myModelRepository: MyModelRepository,
    savedStateHandle: SavedStateHandle
) : ViewModel() {

    val uiState: StateFlow<DetailsUiState> = savedStateHandle
        .getStateFlow<Long?>("id", null)        // ← extract nav arg as StateFlow
        .filterNotNull()                         // ← skip null (before arg arrives)
        .flatMapLatest { id ->
            myModelRepository.observeModelById(id)  // ← switch to this model's Flow
        }
        .map { model ->
            DetailsUiState.Success(
                title = model.title,
                description = model.description
            )
        }
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), DetailsUiState.Loading)
}
```

### Pipeline walkthrough:

| Operator | Input | Output | Why |
|---|---|---|---|
| `getStateFlow<Long?>("id", null)` | — | `StateFlow<Long?>` | Reads the nav argument `id` reactively. Initial value is `null` until navigation passes it. |
| `filterNotNull()` | `Long?` | `Long` | Skips emission while `id` is null. Without this, `observeModelById(null)` would throw. |
| `flatMapLatest { id → ... }` | `Long` | `Flow<MyModel>` | Switches to the Flow for this specific model. If `id` changes (e.g., new navigation), cancels the old Flow and subscribes to the new one. |
| `.map { → Success(...) }` | `MyModel` | `DetailsUiState` | Maps domain model to UI state |
| `.stateIn(...)` | `Flow` | `StateFlow` | Converts to hot StateFlow with initial `Loading` |

### `flatMapLatest` vs `flatMapConcat` vs `switchMap`

`flatMapLatest` cancels the previous inner Flow when a new value arrives.
This is correct for navigation args — if the user navigates to a different
item before the first one loads, we want the old observation cancelled.

---

## HomeViewModel (Wear): simplest case

```kotlin
@HiltViewModel
class HomeViewModel @Inject constructor(
    myModelRepository: MyModelRepository
) : ViewModel() {

    val uiState: StateFlow<HomeUiState> = myModelRepository
        .observeAllModels
        .map<List<MyModel>, HomeUiState>(::Success)
        .stateIn(
            viewModelScope,
            SharingStarted.WhileSubscribed(5000),
            HomeUiState.Loading
        )
}
```

No transformation needed — `MyModel` is used directly in the UI state.
Uses the same `stateIn` + `WhileSubscribed(5000)` pattern.

---

## One-Shot Operations (Bookmark Toggle)

```kotlin
fun bookmark(id: Long, isBookmarked: Boolean) {
    viewModelScope.launch {                          // ← fires and forgets
        myModelRepository.bookmark(id, isBookmarked)  // ← suspending call
    }
}
```

- `viewModelScope.launch` — coroutine tied to ViewModel lifecycle
- `repository.bookmark()` is a `suspend` function that calls `getAndUpdate`
- After the mutation, `MutableStateFlow` emits the new list automatically
- All observers (`observeAllModels`, `observeModelById`) get the updated data

This is the **unidirectional data flow**: user action → ViewModel coroutine → repository mutation → Flow emission → UI recomposition.

---

## Error Handling

This branch takes a **crash-on-error** approach:
- `observeModelById` throws `NoSuchElementException` if the ID isn't found
- No `.catch { }` operators in the ViewModels
- No error state in the `sealed interface` (only `Loading` + `Success`)

This is intentional for a sample — a production app would add `Error` states.

---

## No `combine`, No Custom `Async` Class

Unlike the `main` branch, this branch:
- Does **not** use `combine()` (no filter state, no multiple flows to merge)
- Does **not** have a custom `Async` sealed class
- Does **not** have `CoroutinesModule` with `@IoDispatcher` qualifiers
- Does **not** have `@ApplicationScope` fire-and-forget operations

It's a deliberately minimal implementation focused on modularization structure.
