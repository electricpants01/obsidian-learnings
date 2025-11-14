# StateFlow

## ¿Qué es StateFlow?

**StateFlow** es un observable state-holder flow que emite el estado actual y nuevas actualizaciones a sus collectors. Es similar a LiveData pero con ventajas de Flow.

## Características Principales

- **Hot Flow**: Siempre activo, no necesita collectors
- **State Holder**: Siempre tiene un valor actual
- **Conflation**: Solo emite el último valor
- **Thread-Safe**: Seguro para actualizar desde cualquier thread
- **Distinct Values**: Solo emite cuando el valor realmente cambia

## StateFlow Básico

### 1. Crear StateFlow

```kotlin
class MyViewModel : ViewModel() {
    // MutableStateFlow - para modificar internamente
    private val _uiState = MutableStateFlow(UiState())
    
    // StateFlow - exponer solo lectura
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    // Con valor inicial
    private val _counter = MutableStateFlow(0)
    val counter: StateFlow<Int> = _counter
    
    // StateFlow de tipo nullable
    private val _user = MutableStateFlow<User?>(null)
    val user: StateFlow<User?> = _user.asStateFlow()
}
```

### 2. Actualizar StateFlow

```kotlin
class CounterViewModel : ViewModel() {
    private val _count = MutableStateFlow(0)
    val count: StateFlow<Int> = _count.asStateFlow()
    
    // value - acceso directo
    fun increment() {
        _count.value++
    }
    
    // update - función atómica
    fun incrementAtomic() {
        _count.update { currentValue ->
            currentValue + 1
        }
    }
    
    // updateAndGet - obtener nuevo valor
    fun incrementAndGet(): Int {
        return _count.updateAndGet { it + 1 }
    }
    
    // getAndUpdate - obtener valor anterior
    fun getAndIncrement(): Int {
        return _count.getAndUpdate { it + 1 }
    }
}
```

### 3. Recolectar StateFlow

```kotlin
@Composable
fun CounterScreen(viewModel: CounterViewModel = viewModel()) {
    // collectAsState - para Compose
    val count by viewModel.count.collectAsState()
    
    Text("Count: $count")
}

// En Fragment/Activity
class MyFragment : Fragment() {
    private val viewModel: MyViewModel by viewModels()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // collectAsStateWithLifecycle - respeta lifecycle
        viewLifecycleOwner.lifecycleScope.launch {
            viewModel.uiState.collectAsStateWithLifecycle().collect { state ->
                updateUI(state)
            }
        }
        
        // repeatOnLifecycle - colectar solo cuando está STARTED
        viewLifecycleOwner.lifecycleScope.launch {
            viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    updateUI(state)
                }
            }
        }
    }
}
```

## Patrones de UI State

### 1. Single State Object

```kotlin
data class HomeUiState(
    val isLoading: Boolean = false,
    val items: List<Item> = emptyList(),
    val error: String? = null,
    val selectedItem: Item? = null
)

class HomeViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(HomeUiState())
    val uiState: StateFlow<HomeUiState> = _uiState.asStateFlow()
    
    fun loadItems() {
        _uiState.update { it.copy(isLoading = true) }
        
        viewModelScope.launch {
            try {
                val items = repository.getItems()
                _uiState.update { 
                    it.copy(
                        isLoading = false,
                        items = items,
                        error = null
                    )
                }
            } catch (e: Exception) {
                _uiState.update {
                    it.copy(
                        isLoading = false,
                        error = e.message
                    )
                }
            }
        }
    }
    
    fun selectItem(item: Item) {
        _uiState.update { it.copy(selectedItem = item) }
    }
}
```

### 2. Sealed Class State

```kotlin
sealed interface UiState<out T> {
    object Loading : UiState<Nothing>
    data class Success<T>(val data: T) : UiState<T>
    data class Error(val message: String) : UiState<Nothing>
}

class DataViewModel : ViewModel() {
    private val _dataState = MutableStateFlow<UiState<List<Data>>>(UiState.Loading)
    val dataState: StateFlow<UiState<List<Data>>> = _dataState.asStateFlow()
    
    init {
        loadData()
    }
    
    private fun loadData() {
        viewModelScope.launch {
            _dataState.value = UiState.Loading
            
            try {
                val data = repository.getData()
                _dataState.value = UiState.Success(data)
            } catch (e: Exception) {
                _dataState.value = UiState.Error(e.message ?: "Unknown error")
            }
        }
    }
}

// Uso en Compose
@Composable
fun DataScreen(viewModel: DataViewModel = viewModel()) {
    val state by viewModel.dataState.collectAsState()
    
    when (state) {
        is UiState.Loading -> LoadingIndicator()
        is UiState.Success -> DataList((state as UiState.Success).data)
        is UiState.Error -> ErrorMessage((state as UiState.Error).message)
    }
}
```

### 3. Multiple StateFlows

```kotlin
class ProfileViewModel : ViewModel() {
    private val _user = MutableStateFlow<User?>(null)
    val user: StateFlow<User?> = _user.asStateFlow()
    
    private val _isLoading = MutableStateFlow(false)
    val isLoading: StateFlow<Boolean> = _isLoading.asStateFlow()
    
    private val _error = MutableStateFlow<String?>(null)
    val error: StateFlow<String?> = _error.asStateFlow()
    
    // Combinar múltiples StateFlows
    val uiState: StateFlow<ProfileUiState> = combine(
        user,
        isLoading,
        error
    ) { user, loading, error ->
        ProfileUiState(
            user = user,
            isLoading = loading,
            error = error
        )
    }.stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5000),
        initialValue = ProfileUiState()
    )
}
```

## stateIn - Convertir Flow a StateFlow

```kotlin
class SearchViewModel : ViewModel() {
    private val searchQuery = MutableStateFlow("")
    
    // Convertir Flow a StateFlow
    val searchResults: StateFlow<List<Result>> = searchQuery
        .debounce(300)
        .distinctUntilChanged()
        .flatMapLatest { query ->
            repository.search(query)
        }
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )
    
    fun onSearchQueryChanged(query: String) {
        searchQuery.value = query
    }
}
```

### SharingStarted Strategies

```kotlin
// WhileSubscribed - mantener activo mientras hay subscribers
// stopTimeoutMillis - esperar antes de parar (útil para config changes)
val state1 = flow.stateIn(
    viewModelScope,
    SharingStarted.WhileSubscribed(5000), // Esperar 5s antes de parar
    initialValue
)

// Eagerly - iniciar inmediatamente y nunca parar
val state2 = flow.stateIn(
    viewModelScope,
    SharingStarted.Eagerly,
    initialValue
)

// Lazily - iniciar con primer subscriber y nunca parar
val state3 = flow.stateIn(
    viewModelScope,
    SharingStarted.Lazily,
    initialValue
)
```

## Combine Multiple StateFlows

```kotlin
class DashboardViewModel : ViewModel() {
    private val _user = MutableStateFlow<User?>(null)
    private val _settings = MutableStateFlow(Settings())
    private val _notifications = MutableStateFlow<List<Notification>>(emptyList())
    
    val dashboardState: StateFlow<DashboardState> = combine(
        _user,
        _settings,
        _notifications
    ) { user, settings, notifications ->
        DashboardState(
            user = user,
            settings = settings,
            unreadCount = notifications.count { !it.isRead }
        )
    }.stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5000),
        initialValue = DashboardState()
    )
}
```

## StateFlow vs LiveData

```kotlin
// LiveData
class LiveDataViewModel : ViewModel() {
    private val _data = MutableLiveData<String>()
    val data: LiveData<String> = _data
    
    fun updateData(value: String) {
        _data.value = value // Main thread
        // _data.postValue(value) // Any thread
    }
}

// StateFlow
class StateFlowViewModel : ViewModel() {
    private val _data = MutableStateFlow("")
    val data: StateFlow<String> = _data.asStateFlow()
    
    fun updateData(value: String) {
        _data.value = value // Thread-safe, any thread
    }
}
```

**Ventajas de StateFlow:**
- Thread-safe por defecto
- Operadores de Flow (map, filter, etc.)
- Mejor soporte para testing
- No depende de Android Lifecycle
- Valores iniciales obligatorios (no null)

## Testing StateFlow

```kotlin
class ViewModelTest {
    @Test
    fun `test state flow updates`() = runTest {
        val viewModel = MyViewModel()
        
        // Recolectar valores
        val states = mutableListOf<UiState>()
        val job = launch {
            viewModel.uiState.collect {
                states.add(it)
            }
        }
        
        // Trigger acción
        viewModel.loadData()
        advanceUntilIdle()
        
        // Verificar estados
        assertEquals(UiState.Loading, states[0])
        assertEquals(UiState.Success, states[1])
        
        job.cancel()
    }
    
    @Test
    fun `test state flow with turbine`() = runTest {
        val viewModel = MyViewModel()
        
        viewModel.uiState.test {
            assertEquals(UiState.Initial, awaitItem())
            
            viewModel.loadData()
            assertEquals(UiState.Loading, awaitItem())
            assertEquals(UiState.Success, awaitItem())
        }
    }
}
```

## Mejores Prácticas

### 1. Encapsulation

```kotlin
// ✅ BUENO - Exponer solo lectura
class GoodViewModel : ViewModel() {
    private val _state = MutableStateFlow(State())
    val state: StateFlow<State> = _state.asStateFlow()
}

// ❌ MALO - Exponer MutableStateFlow
class BadViewModel : ViewModel() {
    val state = MutableStateFlow(State()) // Público mutable
}
```

### 2. Initial Values

```kotlin
// ✅ BUENO - Valor inicial significativo
private val _uiState = MutableStateFlow(
    UiState(isLoading = false, data = emptyList())
)

// ❌ MALO - null como valor inicial sin razón
private val _uiState = MutableStateFlow<UiState?>(null)
```

### 3. Update Safety

```kotlin
// ✅ BUENO - usar update para operaciones complejas
fun addItem(item: Item) {
    _state.update { currentState ->
        currentState.copy(
            items = currentState.items + item
        )
    }
}

// ⚠️ RIESGO - modificar directamente puede causar race conditions
fun addItemUnsafe(item: Item) {
    val current = _state.value
    _state.value = current.copy(
        items = current.items + item
    )
}
```

### 4. Lifecycle Awareness en UI

```kotlin
// ✅ BUENO - Usar repeatOnLifecycle
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.state.collect { state ->
            updateUI(state)
        }
    }
}

// ❌ MALO - Sin lifecycle awareness
lifecycleScope.launch {
    viewModel.state.collect { state ->
        updateUI(state) // Puede actualizar UI cuando está en background
    }
}
```

## Patrones Avanzados

### 1. State Reducer Pattern

```kotlin
class ReducerViewModel : ViewModel() {
    private val _state = MutableStateFlow(State())
    val state: StateFlow<State> = _state.asStateFlow()
    
    fun processAction(action: Action) {
        _state.update { currentState ->
            reduce(currentState, action)
        }
    }
    
    private fun reduce(state: State, action: Action): State {
        return when (action) {
            is Action.LoadData -> state.copy(isLoading = true)
            is Action.DataLoaded -> state.copy(
                isLoading = false,
                data = action.data
            )
            is Action.Error -> state.copy(
                isLoading = false,
                error = action.message
            )
        }
    }
}

sealed class Action {
    object LoadData : Action()
    data class DataLoaded(val data: List<Item>) : Action()
    data class Error(val message: String) : Action()
}
```

### 2. Debounced Updates

```kotlin
class SearchViewModel : ViewModel() {
    private val _rawQuery = MutableStateFlow("")
    
    val debouncedQuery: StateFlow<String> = _rawQuery
        .debounce(300)
        .stateIn(
            viewModelScope,
            SharingStarted.WhileSubscribed(),
            ""
        )
    
    fun onQueryChange(query: String) {
        _rawQuery.value = query
    }
}
```

## Recursos

- [StateFlow Documentation](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/)
- [StateFlow vs LiveData](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow)
- [Migrating to StateFlow](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow#livedata)
- [State in Compose](https://developer.android.com/jetpack/compose/state)

---
*StateFlow es el state holder moderno para ViewModels en Android*
