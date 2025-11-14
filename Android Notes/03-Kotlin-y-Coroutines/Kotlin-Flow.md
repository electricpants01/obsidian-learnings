# Kotlin Flow

## ¿Qué es Flow?

**Flow** es una API de Kotlin para programación reactiva que emite múltiples valores secuencialmente. Es similar a Sequences pero con soporte para operaciones asíncronas.

## Flow Básico

### 1. Crear Flows

```kotlin
// Flow simple
fun simpleFlow(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // Operación asíncrona
        emit(i) // Emitir valor
    }
}

// Flow desde colección
val numbersFlow: Flow<Int> = listOf(1, 2, 3).asFlow()

// Flow desde rango
val rangeFlow: Flow<Int> = (1..5).asFlow()

// flowOf - crear flow de valores específicos
val specificFlow = flowOf("A", "B", "C")

// channelFlow - para casos complejos
fun complexFlow() = channelFlow {
    launch {
        send(1)
        delay(100)
        send(2)
    }
}
```

### 2. Recolectar Flows

```kotlin
// Básico - collect
viewModelScope.launch {
    simpleFlow().collect { value ->
        println("Received: $value")
    }
}

// collectLatest - cancela colección anterior si llega nuevo valor
viewModelScope.launch {
    searchQuery.collectLatest { query ->
        // Si llega nueva query, cancela búsqueda anterior
        val results = searchRepository.search(query)
        _searchResults.value = results
    }
}

// collectIndexed - con índice
flow.collectIndexed { index, value ->
    println("$index: $value")
}

// toList - convertir a lista (suspending)
val list = flow.toList()

// toSet - convertir a set
val set = flow.toSet()

// first - primer elemento
val first = flow.first()

// last - último elemento  
val last = flow.last()

// single - único elemento (error si hay más)
val single = flow.single()
```

## Operadores de Flow

### 1. Operadores de Transformación

```kotlin
// map - transformar cada valor
flow.map { value -> value * 2 }
    .collect { println(it) }

// mapNotNull - filtrar nulls
flow.mapNotNull { value -> 
    if (value > 0) value else null
}

// transform - transformación personalizada
flow.transform { value ->
    emit(value)
    emit(value * 2)
}

// scan - acumulador que emite valores intermedios
(1..5).asFlow()
    .scan(0) { acc, value -> acc + value }
    .collect { println(it) } // 0, 1, 3, 6, 10, 15

// flatMapConcat - concatenar flows secuencialmente
flow.flatMapConcat { value ->
    flow { 
        emit("$value-A")
        emit("$value-B")
    }
}

// flatMapMerge - merge flows concurrentemente
flow.flatMapMerge { value ->
    flow {
        delay(100)
        emit(value * 2)
    }
}

// flatMapLatest - cancelar anterior
searchQuery.flatMapLatest { query ->
    searchRepository.search(query)
}
```

### 2. Operadores de Filtrado

```kotlin
// filter - filtrar valores
flow.filter { it > 0 }

// filterNot - inverso de filter
flow.filterNot { it < 0 }

// filterIsInstance - filtrar por tipo
mixedFlow.filterIsInstance<String>()

// filterNotNull - filtrar nulls
flow.filterNotNull()

// take - tomar primeros n elementos
flow.take(5)

// takeWhile - tomar mientras condición
flow.takeWhile { it < 10 }

// drop - saltar primeros n elementos
flow.drop(2)

// dropWhile - saltar mientras condición
flow.dropWhile { it < 5 }

// distinctUntilChanged - emitir solo cuando cambia
flow.distinctUntilChanged()

// distinctUntilChangedBy - por selector
users.distinctUntilChangedBy { it.id }

// debounce - emitir solo después de tiempo sin nuevos valores
searchQuery.debounce(300) // 300ms
```

### 3. Operadores de Combinación

```kotlin
// combine - combinar últimos valores de múltiples flows
val combined = combine(flow1, flow2, flow3) { a, b, c ->
    "$a-$b-$c"
}

// zip - combinar valores por posición
val zipped = flow1.zip(flow2) { a, b -> "$a-$b" }

// merge - merge múltiples flows
val merged = merge(flow1, flow2, flow3)

// flattenConcat - concatenar flows
flowOf(flow1, flow2).flattenConcat()

// flattenMerge - merge flows concurrentemente
flowOf(flow1, flow2).flattenMerge()
```

### 4. Operadores de Terminal

```kotlin
// reduce - acumular a valor final
val sum = (1..10).asFlow().reduce { acc, value -> acc + value }

// fold - como reduce pero con valor inicial
val result = flow.fold(0) { acc, value -> acc + value }

// count - contar elementos
val count = flow.count()

// any - si alguno cumple
val hasPositive = flow.any { it > 0 }

// all - si todos cumplen
val allPositive = flow.all { it > 0 }

// none - si ninguno cumple
val noneNegative = flow.none { it < 0 }
```

## Context y Dispatchers

```kotlin
// flowOn - cambiar contexto de ejecución upstream
fun getData(): Flow<Data> = flow {
    // Ejecuta en Dispatchers.IO
    val data = heavyOperation()
    emit(data)
}.flowOn(Dispatchers.IO)

// Recolectar en Main
viewModelScope.launch(Dispatchers.Main) {
    getData().collect { data ->
        // UI update en Main
        _uiState.value = data
    }
}

// conflate - saltar valores intermedios si el collector es lento
flow.conflate()
    .collect { value ->
        delay(300) // Lento
        println(value)
    }

// buffer - buffer de valores
flow.buffer(capacity = 10)

// flowOn con múltiples operadores
flow
    .map { /* transform */ }
    .flowOn(Dispatchers.Default)
    .filter { /* filter */ }
    .flowOn(Dispatchers.IO)
```

## Exception Handling

```kotlin
// try-catch en collect
viewModelScope.launch {
    flow.collect { value ->
        try {
            processValue(value)
        } catch (e: Exception) {
            handleError(e)
        }
    }
}

// catch - manejar excepciones upstream
flow
    .map { value -> value.toInt() }
    .catch { e -> 
        emit(-1) // Valor por defecto
        println("Error: ${e.message}")
    }
    .collect { value ->
        println(value)
    }

// retry - reintentar en error
flow
    .retry(retries = 3) { cause ->
        cause is IOException
    }
    .catch { e ->
        emit(emptyList())
    }
    .collect { }

// retryWhen - retry con lógica personalizada
flow.retryWhen { cause, attempt ->
    if (cause is IOException && attempt < 3) {
        delay(1000 * attempt)
        true
    } else {
        false
    }
}

// onCompletion - ejecutar al completar
flow
    .onCompletion { cause ->
        if (cause != null) {
            println("Flow completed with exception: $cause")
        } else {
            println("Flow completed successfully")
        }
    }
    .collect { }
```

## Operadores de Timing

```kotlin
// delay en cada emisión
flow {
    repeat(5) { i ->
        emit(i)
        delay(100)
    }
}

// onStart - ejecutar antes de comenzar
flow
    .onStart { emit(-1) } // Emitir valor inicial
    .collect { }

// onEach - ejecutar para cada valor
flow
    .onEach { delay(100) }
    .collect { }

// sample - emitir último valor cada período
flow.sample(1000) // Cada segundo

// timeout - timeout para la colección
withTimeout(5000) {
    flow.collect { }
}
```

## SharedFlow vs StateFlow

Ver archivos separados: [StateFlow.md](StateFlow.md) y [SharedFlow.md](SharedFlow.md)

## Patrones Comunes

### 1. Repository Pattern con Flow

```kotlin
class UserRepository(
    private val api: UserApi,
    private val dao: UserDao
) {
    // Flow desde Room
    fun getUsers(): Flow<List<User>> = dao.observeUsers()
    
    // Flow desde API con cache
    fun getUsersWithRefresh(): Flow<Resource<List<User>>> = flow {
        emit(Resource.Loading())
        
        // Emitir cache primero
        val cached = dao.getUsers()
        if (cached.isNotEmpty()) {
            emit(Resource.Success(cached))
        }
        
        try {
            // Fetch de red
            val users = api.getUsers()
            dao.insertUsers(users)
            emit(Resource.Success(users))
        } catch (e: Exception) {
            emit(Resource.Error(e.message ?: "Unknown error"))
        }
    }
    
    // Combinar múltiples fuentes
    fun getCombinedData(): Flow<CombinedData> = combine(
        dao.observeUsers(),
        dao.observePosts(),
        settingsFlow
    ) { users, posts, settings ->
        CombinedData(users, posts, settings)
    }
}
```

### 2. Search con Debounce

```kotlin
class SearchViewModel : ViewModel() {
    private val _searchQuery = MutableStateFlow("")
    val searchQuery: StateFlow<String> = _searchQuery
    
    val searchResults: StateFlow<List<SearchResult>> = searchQuery
        .debounce(300) // Esperar 300ms sin cambios
        .distinctUntilChanged()
        .filter { it.length >= 3 } // Mínimo 3 caracteres
        .flatMapLatest { query ->
            searchRepository.search(query)
                .catch { emit(emptyList()) }
        }
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )
    
    fun onSearchQueryChanged(query: String) {
        _searchQuery.value = query
    }
}
```

### 3. Paginación con Flow

```kotlin
class PaginatedRepository {
    private var currentPage = 0
    
    fun loadPages(): Flow<List<Item>> = flow {
        while (true) {
            val items = api.getPage(currentPage)
            emit(items)
            currentPage++
            
            if (items.isEmpty()) break
        }
    }
}

// ViewModel
class ListViewModel : ViewModel() {
    val items = repository.loadPages()
        .scan(emptyList<Item>()) { accumulator, newItems ->
            accumulator + newItems
        }
        .stateIn(
            viewModelScope,
            SharingStarted.Lazily,
            emptyList()
        )
}
```

### 4. Polling

```kotlin
fun pollUpdates(interval: Long): Flow<Data> = flow {
    while (true) {
        val data = api.fetchData()
        emit(data)
        delay(interval)
    }
}

// Con retry y error handling
fun pollWithRetry(): Flow<Data> = flow {
    while (true) {
        try {
            val data = api.fetchData()
            emit(data)
            delay(5000)
        } catch (e: Exception) {
            // Continuar polling incluso con errores
            delay(5000)
        }
    }
}
```

## Testing de Flows

```kotlin
class FlowTest {
    @Test
    fun `test flow emissions`() = runTest {
        val flow = flow {
            emit(1)
            emit(2)
            emit(3)
        }
        
        val result = flow.toList()
        assertEquals(listOf(1, 2, 3), result)
    }
    
    @Test
    fun `test flow with turbine`() = runTest {
        flow {
            emit(1)
            emit(2)
            emit(3)
        }.test {
            assertEquals(1, awaitItem())
            assertEquals(2, awaitItem())
            assertEquals(3, awaitItem())
            awaitComplete()
        }
    }
    
    @Test
    fun `test flow errors`() = runTest {
        val flow = flow<Int> {
            throw RuntimeException("Error")
        }
        
        assertFailsWith<RuntimeException> {
            flow.collect()
        }
    }
}
```

## Mejores Prácticas

### 1. Cold vs Hot Flows

```kotlin
// ❌ MALO - crear flow en cada colección
val flow = flow {
    println("Flow started")
    emit(1)
}

// Cada collect inicia el flow de nuevo
flow.collect { } // Prints "Flow started"
flow.collect { } // Prints "Flow started" again

// ✅ BUENO - usar SharedFlow/StateFlow para hot flows
val sharedFlow = flow.shareIn(
    scope = viewModelScope,
    started = SharingStarted.Lazily,
    replay = 0
)
```

### 2. Evitar Memory Leaks

```kotlin
// ❌ MALO - collect sin scope
GlobalScope.launch {
    flow.collect { }
}

// ✅ BUENO - usar lifecycle scope apropiado
viewModelScope.launch {
    flow.collect { }
}

// ✅ MEJOR - usar stateIn con lifecycle
val uiState = flow.stateIn(
    scope = viewModelScope,
    started = SharingStarted.WhileSubscribed(5000),
    initialValue = InitialState
)
```

### 3. Cancelación

```kotlin
// El flow respeta cancelación automáticamente
viewModelScope.launch {
    try {
        flow.collect { value ->
            processValue(value)
        }
    } catch (e: CancellationException) {
        // Limpieza si es necesario
        throw e // Re-throw para propagar cancelación
    }
}
```

## Recursos

- [Kotlin Flow Documentation](https://kotlinlang.org/docs/flow.html)
- [Flow on Android](https://developer.android.com/kotlin/flow)
- [Flow API Reference](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/)
- [Advanced Flow](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow)

---
*Flow es fundamental para programación reactiva en Android moderno*
