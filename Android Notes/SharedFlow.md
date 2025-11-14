# SharedFlow

## ¿Qué es SharedFlow?

**SharedFlow** es un hot flow que puede tener múltiples collectors y emite valores a todos ellos. A diferencia de StateFlow, no mantiene un valor actual y puede configurar replay y buffer.

## Características Principales

- **Hot Flow**: Emite valores incluso sin collectors
- **Multiple Collectors**: Todos reciben los mismos valores
- **Configurable Replay**: Puede reemitir valores anteriores
- **Buffer Strategy**: Control sobre qué hacer cuando buffer está lleno
- **No State Holder**: No mantiene estado actual (a menos que replay > 0)

## SharedFlow vs StateFlow

| Característica | SharedFlow | StateFlow |
|---------------|------------|-----------|
| Valor inicial | No requiere | Siempre requiere |
| Replay | Configurable | Siempre 1 |
| Distinct values | No | Sí |
| Uso típico | Eventos | Estado |

## SharedFlow Básico

### 1. Crear SharedFlow

```kotlin
class EventsViewModel : ViewModel() {
    // MutableSharedFlow - para emitir
    private val _events = MutableSharedFlow<Event>()
    
    // SharedFlow - exponer solo lectura
    val events: SharedFlow<Event> = _events.asSharedFlow()
    
    // Con configuración
    private val _messages = MutableSharedFlow<String>(
        replay = 0,           // Cantidad de valores a reemitir
        extraBufferCapacity = 64,  // Capacidad adicional del buffer
        onBufferOverflow = BufferOverflow.DROP_OLDEST
    )
    
    fun sendEvent(event: Event) {
        viewModelScope.launch {
            _events.emit(event)
        }
    }
}
```

### 2. Replay y Buffer

```kotlin
// Sin replay - solo collectors activos reciben
val sharedFlow1 = MutableSharedFlow<Int>(replay = 0)

// Replay 1 - nuevo collector recibe último valor
val sharedFlow2 = MutableSharedFlow<Int>(replay = 1)

// Replay ilimitado - todos los valores
val sharedFlow3 = MutableSharedFlow<Int>(
    replay = Int.MAX_VALUE,
    onBufferOverflow = BufferOverflow.DROP_OLDEST
)

// Buffer extra
val sharedFlow4 = MutableSharedFlow<Int>(
    replay = 1,
    extraBufferCapacity = 10 // 10 valores adicionales en buffer
)
```

### 3. Buffer Overflow Strategies

```kotlin
// SUSPEND - suspender emisor cuando buffer está lleno (default)
MutableSharedFlow<Int>(
    extraBufferCapacity = 10,
    onBufferOverflow = BufferOverflow.SUSPEND
)

// DROP_OLDEST - eliminar valor más antiguo
MutableSharedFlow<Int>(
    extraBufferCapacity = 10,
    onBufferOverflow = BufferOverflow.DROP_OLDEST
)

// DROP_LATEST - eliminar valor más reciente
MutableSharedFlow<Int>(
    extraBufferCapacity = 10,
    onBufferOverflow = BufferOverflow.DROP_LATEST
)
```

## Patrones de Uso - Eventos

### 1. One-Time Events

```kotlin
sealed class UiEvent {
    data class ShowSnackbar(val message: String) : UiEvent()
    data class NavigateTo(val route: String) : UiEvent()
    object ShowLoading : UiEvent()
}

class MyViewModel : ViewModel() {
    private val _events = MutableSharedFlow<UiEvent>()
    val events: SharedFlow<UiEvent> = _events.asSharedFlow()
    
    fun onSaveClicked() {
        viewModelScope.launch {
            try {
                repository.save()
                _events.emit(UiEvent.ShowSnackbar("Saved successfully"))
            } catch (e: Exception) {
                _events.emit(UiEvent.ShowSnackbar("Error: ${e.message}"))
            }
        }
    }
    
    fun navigateToDetails(id: String) {
        viewModelScope.launch {
            _events.emit(UiEvent.NavigateTo("details/$id"))
        }
    }
}

// Uso en Compose
@Composable
fun MyScreen(viewModel: MyViewModel = viewModel()) {
    val snackbarHostState = remember { SnackbarHostState() }
    val navController = rememberNavController()
    
    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is UiEvent.ShowSnackbar -> {
                    snackbarHostState.showSnackbar(event.message)
                }
                is UiEvent.NavigateTo -> {
                    navController.navigate(event.route)
                }
                UiEvent.ShowLoading -> {
                    // Show loading
                }
            }
        }
    }
    
    Scaffold(
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { padding ->
        // Content
    }
}
```

### 2. Error Events

```kotlin
class ErrorViewModel : ViewModel() {
    private val _errorEvents = MutableSharedFlow<ErrorEvent>()
    val errorEvents: SharedFlow<ErrorEvent> = _errorEvents.asSharedFlow()
    
    suspend fun handleError(error: Throwable) {
        val errorEvent = when (error) {
            is NetworkException -> ErrorEvent.NetworkError
            is AuthException -> ErrorEvent.AuthError
            else -> ErrorEvent.UnknownError(error.message ?: "Unknown error")
        }
        _errorEvents.emit(errorEvent)
    }
}

sealed class ErrorEvent {
    object NetworkError : ErrorEvent()
    object AuthError : ErrorEvent()
    data class UnknownError(val message: String) : ErrorEvent()
}
```

### 3. Navigation Events

```kotlin
sealed class NavigationEvent {
    object NavigateBack : NavigationEvent()
    data class NavigateTo(val route: String) : NavigationEvent()
    data class NavigateWithResult<T>(val route: String, val result: T) : NavigationEvent()
}

class NavigationViewModel : ViewModel() {
    private val _navigationEvents = MutableSharedFlow<NavigationEvent>()
    val navigationEvents: SharedFlow<NavigationEvent> = _navigationEvents.asSharedFlow()
    
    fun navigateBack() {
        viewModelScope.launch {
            _navigationEvents.emit(NavigationEvent.NavigateBack)
        }
    }
    
    fun navigateToDetails(id: String) {
        viewModelScope.launch {
            _navigationEvents.emit(NavigationEvent.NavigateTo("details/$id"))
        }
    }
}
```

## tryEmit vs emit

```kotlin
class EmitViewModel : ViewModel() {
    private val _events = MutableSharedFlow<Event>(
        extraBufferCapacity = 10,
        onBufferOverflow = BufferOverflow.DROP_OLDEST
    )
    
    // tryEmit - no suspende, retorna boolean
    fun sendEventNonSuspending(event: Event) {
        val emitted = _events.tryEmit(event)
        if (!emitted) {
            // Evento no fue emitido (buffer lleno)
            Log.w("EventsViewModel", "Event dropped")
        }
    }
    
    // emit - suspende si es necesario
    fun sendEvent(event: Event) {
        viewModelScope.launch {
            _events.emit(event) // Puede suspender si buffer está lleno
        }
    }
}
```

## shareIn - Convertir Flow a SharedFlow

```kotlin
class RepositoryViewModel : ViewModel() {
    // Convertir cold flow a hot shared flow
    val updates: SharedFlow<Update> = repository.getUpdates()
        .shareIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            replay = 1
        )
    
    // Con procesamiento
    val processedData: SharedFlow<ProcessedData> = repository.getData()
        .map { processData(it) }
        .shareIn(
            scope = viewModelScope,
            started = SharingStarted.Eagerly,
            replay = 0
        )
}
```

### SharingStarted Strategies

```kotlin
// WhileSubscribed - activo mientras hay subscribers
val flow1 = flow.shareIn(
    viewModelScope,
    SharingStarted.WhileSubscribed(5000), // 5s timeout
    replay = 0
)

// Eagerly - iniciar inmediatamente
val flow2 = flow.shareIn(
    viewModelScope,
    SharingStarted.Eagerly,
    replay = 1
)

// Lazily - iniciar con primer subscriber
val flow3 = flow.shareIn(
    viewModelScope,
    SharingStarted.Lazily,
    replay = 1
)
```

## Subscription Count

```kotlin
class AnalyticsViewModel : ViewModel() {
    private val _events = MutableSharedFlow<Event>()
    val events: SharedFlow<Event> = _events.asSharedFlow()
    
    init {
        viewModelScope.launch {
            _events.subscriptionCount.collect { count ->
                Log.d("Analytics", "Active collectors: $count")
                if (count > 0) {
                    // Hay collectors activos
                } else {
                    // No hay collectors
                }
            }
        }
    }
}
```

## Combining SharedFlows

```kotlin
class CombinedEventsViewModel : ViewModel() {
    private val _eventSource1 = MutableSharedFlow<Event>()
    private val _eventSource2 = MutableSharedFlow<Event>()
    
    // Merge múltiples SharedFlows
    val allEvents: SharedFlow<Event> = merge(
        _eventSource1,
        _eventSource2
    ).shareIn(
        scope = viewModelScope,
        started = SharingStarted.Lazily,
        replay = 0
    )
}
```

## Testing SharedFlow

```kotlin
class SharedFlowTest {
    @Test
    fun `test shared flow emissions`() = runTest {
        val viewModel = EventsViewModel()
        val events = mutableListOf<Event>()
        
        val job = launch {
            viewModel.events.collect {
                events.add(it)
            }
        }
        
        viewModel.sendEvent(Event.A)
        viewModel.sendEvent(Event.B)
        advanceUntilIdle()
        
        assertEquals(2, events.size)
        assertEquals(Event.A, events[0])
        assertEquals(Event.B, events[1])
        
        job.cancel()
    }
    
    @Test
    fun `test shared flow with turbine`() = runTest {
        val viewModel = EventsViewModel()
        
        viewModel.events.test {
            viewModel.sendEvent(Event.A)
            assertEquals(Event.A, awaitItem())
            
            viewModel.sendEvent(Event.B)
            assertEquals(Event.B, awaitItem())
            
            cancelAndIgnoreRemainingEvents()
        }
    }
    
    @Test
    fun `test replay behavior`() = runTest {
        val sharedFlow = MutableSharedFlow<Int>(replay = 2)
        
        // Emitir valores antes de collector
        sharedFlow.emit(1)
        sharedFlow.emit(2)
        sharedFlow.emit(3)
        
        // Nuevo collector recibe últimos 2 valores
        val received = mutableListOf<Int>()
        val job = launch {
            sharedFlow.take(2).collect { received.add(it) }
        }
        
        advanceUntilIdle()
        assertEquals(listOf(2, 3), received) // Solo últimos 2
        
        job.cancel()
    }
}
```

## Patrones Avanzados

### 1. Event Bus Pattern

```kotlin
// Event Bus global
object EventBus {
    private val _events = MutableSharedFlow<AppEvent>()
    val events: SharedFlow<AppEvent> = _events.asSharedFlow()
    
    suspend fun emit(event: AppEvent) {
        _events.emit(event)
    }
}

sealed class AppEvent {
    object UserLoggedOut : AppEvent()
    data class ThemeChanged(val theme: Theme) : AppEvent()
    data class LanguageChanged(val language: String) : AppEvent()
}

// Uso
class SomeViewModel : ViewModel() {
    init {
        viewModelScope.launch {
            EventBus.events.collect { event ->
                when (event) {
                    AppEvent.UserLoggedOut -> handleLogout()
                    is AppEvent.ThemeChanged -> applyTheme(event.theme)
                    is AppEvent.LanguageChanged -> updateLanguage(event.language)
                }
            }
        }
    }
}
```

### 2. Debounced Events

```kotlin
class DebouncedEventsViewModel : ViewModel() {
    private val _clickEvents = MutableSharedFlow<Unit>(
        extraBufferCapacity = 10,
        onBufferOverflow = BufferOverflow.DROP_OLDEST
    )
    
    val processedClicks: SharedFlow<Unit> = _clickEvents
        .debounce(300)
        .shareIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(),
            replay = 0
        )
    
    fun onClick() {
        _clickEvents.tryEmit(Unit)
    }
}
```

### 3. Throttled Events

```kotlin
class ThrottledEventsViewModel : ViewModel() {
    private val _events = MutableSharedFlow<Event>()
    
    // Throttle - emitir máximo cada N ms
    val throttledEvents: SharedFlow<Event> = _events
        .sample(1000) // Máximo uno por segundo
        .shareIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(),
            replay = 0
        )
}
```

## Mejores Prácticas

### 1. No usar para Estado

```kotlin
// ❌ MALO - usar SharedFlow para estado
class BadViewModel : ViewModel() {
    private val _state = MutableSharedFlow<UiState>()
    val state: SharedFlow<UiState> = _state.asSharedFlow()
}

// ✅ BUENO - usar StateFlow para estado
class GoodViewModel : ViewModel() {
    private val _state = MutableStateFlow(UiState())
    val state: StateFlow<UiState> = _state.asStateFlow()
}
```

### 2. Eventos One-Time

```kotlin
// ✅ BUENO - SharedFlow para eventos que no deben repetirse
class GoodViewModel : ViewModel() {
    private val _events = MutableSharedFlow<Event>()
    val events: SharedFlow<Event> = _events.asSharedFlow()
    
    fun triggerEvent() {
        viewModelScope.launch {
            _events.emit(Event.SomeEvent)
        }
    }
}
```

### 3. Manejo de Config Changes

```kotlin
// En Compose con LaunchedEffect
@Composable
fun MyScreen(viewModel: MyViewModel = viewModel()) {
    val lifecycleOwner = LocalLifecycleOwner.current
    
    LaunchedEffect(lifecycleOwner) {
        lifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
            viewModel.events.collect { event ->
                handleEvent(event)
            }
        }
    }
}
```

### 4. Replay Apropiado

```kotlin
// replay = 0 para eventos únicos
private val _navigationEvents = MutableSharedFlow<NavigationEvent>(replay = 0)

// replay = 1 para último valor importante
private val _toastMessages = MutableSharedFlow<String>(replay = 1)

// replay > 1 solo si realmente necesitas history
private val _analytics = MutableSharedFlow<AnalyticsEvent>(
    replay = 100,
    onBufferOverflow = BufferOverflow.DROP_OLDEST
)
```

## Channel vs SharedFlow

```kotlin
// Channel - single consumer
val channel = Channel<Event>()

// SharedFlow - multiple consumers
val sharedFlow = MutableSharedFlow<Event>()

// Convertir Channel a SharedFlow
val channelFlow = channel.receiveAsFlow().shareIn(
    scope = viewModelScope,
    started = SharingStarted.Lazily,
    replay = 0
)
```

## Recursos

- [SharedFlow Documentation](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/)
- [StateFlow and SharedFlow](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow)
- [Migrating from LiveData](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow#livedata)
- [Events vs State](https://medium.com/androiddevelopers/viewmodel-one-off-event-antipatterns-16a1da869b95)

---
*SharedFlow es ideal para eventos one-time que múltiples observers necesitan recibir*
