# Kotlin Coroutines

## ¿Qué son las Coroutines?

Las **Coroutines** son una característica de Kotlin para programación asíncrona que permite escribir código asíncrono de manera secuencial y legible, sin callbacks.

## Características Principales

- **Suspendibles**: Pueden pausarse y reanudarse
- **Ligeras**: Menos overhead que threads
- **Estructuradas**: Organized en jerarquías (scope)
- **Cancelables**: Se pueden cancelar cuando ya no son necesarias
- **Non-blocking**: No bloquean threads

## Setup

```kotlin
// En build.gradle
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")
}
```

## Coroutines Básicas

### 1. Función Suspendible

```kotlin
// suspend - marca función como suspendible
suspend fun fetchUser(): User {
    delay(1000) // Suspende por 1 segundo
    return User("John", 25)
}

// Solo puede llamarse desde otra suspend function o coroutine
suspend fun loadUserData() {
    val user = fetchUser() // OK
    println(user)
}

// Error: suspend function solo en coroutine
fun regularFunction() {
    val user = fetchUser() // ❌ ERROR
}
```

### 2. Launch - Fire and Forget

```kotlin
// launch - inicia coroutine, no retorna resultado
fun loadData() {
    viewModelScope.launch {
        val data = fetchData() // suspend function
        updateUI(data)
    }
}

// Con manejo de errores
viewModelScope.launch {
    try {
        val data = repository.getData()
        _uiState.value = UiState.Success(data)
    } catch (e: Exception) {
        _uiState.value = UiState.Error(e.message)
    }
}

// launch retorna Job
val job = viewModelScope.launch {
    // Work
}

// Cancelar job
job.cancel()

// Esperar a que termine
job.join()
```

### 3. Async - Con Resultado

```kotlin
// async - retorna Deferred<T>
suspend fun loadUserWithPosts(): UserWithPosts {
    return coroutineScope {
        // Ejecutar en paralelo
        val userDeferred = async { fetchUser() }
        val postsDeferred = async { fetchPosts() }
        
        // Esperar resultados
        val user = userDeferred.await()
        val posts = postsDeferred.await()
        
        UserWithPosts(user, posts)
    }
}

// Múltiples operaciones en paralelo
suspend fun loadDashboard(): Dashboard {
    return coroutineScope {
        val users = async { repository.getUsers() }
        val posts = async { repository.getPosts() }
        val comments = async { repository.getComments() }
        
        Dashboard(
            users = users.await(),
            posts = posts.await(),
            comments = comments.await()
        )
    }
}
```

### 4. withContext - Cambiar Contexto

```kotlin
suspend fun saveToDatabase(data: Data) {
    // Cambiar a IO dispatcher
    withContext(Dispatchers.IO) {
        database.save(data)
    }
    // Automáticamente vuelve al dispatcher anterior
}

// Ejemplo completo
suspend fun loadAndSaveData() {
    val data = withContext(Dispatchers.IO) {
        // Operación en IO
        api.fetchData()
    }
    
    // Procesar en Default
    val processed = withContext(Dispatchers.Default) {
        processData(data)
    }
    
    // Actualizar UI en Main
    withContext(Dispatchers.Main) {
        updateUI(processed)
    }
}
```

## Builders de Coroutines

### 1. runBlocking

```kotlin
// runBlocking - bloquea thread actual (solo para testing/main)
fun main() = runBlocking {
    launch {
        delay(1000)
        println("World")
    }
    println("Hello")
}

// ❌ NO usar en producción (bloquea UI thread)
fun loadDataBlocking() = runBlocking {
    val data = repository.getData()
}

// ✅ Solo para tests
@Test
fun testSomething() = runBlocking {
    val result = repository.getData()
    assertEquals(expected, result)
}
```

### 2. coroutineScope

```kotlin
// coroutineScope - crea scope que espera a todos sus hijos
suspend fun loadAllData() = coroutineScope {
    launch { loadUsers() }
    launch { loadPosts() }
    // Espera a que ambos terminen antes de retornar
}

// Si alguno falla, cancela todos
suspend fun loadDataWithError() = coroutineScope {
    launch { loadUsers() }
    launch { throw Exception() } // Cancela loadUsers también
}
```

### 3. supervisorScope

```kotlin
// supervisorScope - fallo de un hijo no cancela otros
suspend fun loadDataSafely() = supervisorScope {
    val users = async { loadUsers() }
    val posts = async { 
        throw Exception() // No cancela users
    }
    
    try {
        UserWithPosts(users.await(), posts.await())
    } catch (e: Exception) {
        UserWithPosts(users.await(), emptyList())
    }
}
```

## Delay vs Thread.sleep

```kotlin
// ❌ MALO - bloquea thread
suspend fun bad() {
    Thread.sleep(1000)  // Bloquea thread
}

// ✅ BUENO - suspende coroutine
suspend fun good() {
    delay(1000)  // Solo suspende, libera thread
}
```

## Cancelación

### 1. Cancelación Básica

```kotlin
class MyViewModel : ViewModel() {
    private var job: Job? = null
    
    fun startWork() {
        job = viewModelScope.launch {
            repeat(100) { i ->
                delay(1000)
                println("Working $i")
            }
        }
    }
    
    fun stopWork() {
        job?.cancel()
    }
}
```

### 2. Coroutine Cancelable

```kotlin
suspend fun doWork() {
    withContext(Dispatchers.Default) {
        repeat(100) { i ->
            // Verificar si fue cancelada
            ensureActive()
            // o
            if (!isActive) return@withContext
            
            // Trabajo
            processItem(i)
        }
    }
}

// yield() - punto de cancelación
suspend fun doLongWork() {
    repeat(100) { i ->
        yield() // Permite cancelación
        processItem(i)
    }
}
```

### 3. Cleanup en Cancelación

```kotlin
suspend fun workWithCleanup() {
    try {
        // Trabajo
        doWork()
    } finally {
        // Cleanup - siempre se ejecuta
        withContext(NonCancellable) {
            // Cleanup que debe completarse
            cleanup()
        }
    }
}

// invokeOnCompletion
val job = launch {
    // Work
}

job.invokeOnCompletion { cause ->
    if (cause is CancellationException) {
        println("Cancelled")
    } else if (cause != null) {
        println("Failed: $cause")
    } else {
        println("Completed successfully")
    }
}
```

## Timeout

```kotlin
// withTimeout - lanza TimeoutCancellationException
suspend fun fetchWithTimeout(): Data? {
    return try {
        withTimeout(5000) {
            api.fetchData()
        }
    } catch (e: TimeoutCancellationException) {
        null
    }
}

// withTimeoutOrNull - retorna null en timeout
suspend fun fetchOrNull(): Data? {
    return withTimeoutOrNull(5000) {
        api.fetchData()
    }
}
```

## Patrones Comunes

### 1. Retry Pattern

```kotlin
suspend fun <T> retryIO(
    times: Int = 3,
    initialDelay: Long = 100,
    maxDelay: Long = 1000,
    factor: Double = 2.0,
    block: suspend () -> T
): T {
    var currentDelay = initialDelay
    repeat(times - 1) {
        try {
            return block()
        } catch (e: Exception) {
            // Esperar antes de retry
            delay(currentDelay)
            currentDelay = (currentDelay * factor).toLong().coerceAtMost(maxDelay)
        }
    }
    return block() // Último intento
}

// Uso
val data = retryIO {
    api.fetchData()
}
```

### 2. Sequential vs Parallel

```kotlin
// Sequential - uno después del otro
suspend fun sequential() {
    val user = fetchUser()      // Espera 1s
    val posts = fetchPosts()    // Espera 1s
    // Total: 2s
}

// Parallel - al mismo tiempo
suspend fun parallel() = coroutineScope {
    val user = async { fetchUser() }   // 1s
    val posts = async { fetchPosts() } // 1s
    user.await() to posts.await()
    // Total: 1s
}
```

### 3. Cold Flow desde Coroutine

```kotlin
fun observeData(): Flow<Data> = flow {
    while (true) {
        val data = fetchData()
        emit(data)
        delay(5000)
    }
}
```

### 4. ViewModel Pattern

```kotlin
class MyViewModel : ViewModel() {
    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
    
    init {
        loadData()
    }
    
    private fun loadData() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            
            try {
                val data = repository.getData()
                _uiState.value = UiState.Success(data)
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e.message ?: "Unknown error")
            }
        }
    }
    
    fun refresh() {
        loadData()
    }
}
```

## Testing Coroutines

```kotlin
// runTest - para testing
@Test
fun testCoroutine() = runTest {
    val result = repository.getData()
    assertEquals(expected, result)
}

// advanceUntilIdle - avanzar tiempo virtual
@Test
fun testWithDelay() = runTest {
    var result = 0
    launch {
        delay(1000)
        result = 1
    }
    
    advanceUntilIdle() // Avanza el tiempo virtual
    assertEquals(1, result)
}

// TestDispatchers
@Test
fun testWithDispatcher() = runTest {
    val testDispatcher = StandardTestDispatcher(testScheduler)
    
    withContext(testDispatcher) {
        // Test
    }
}
```

## Dispatchers

Ver archivo separado: [Dispatchers.md](Dispatchers.md)

## Structured Concurrency

Ver archivo separado: [Structured-Concurrency.md](Structured-Concurrency.md)

## Exception Handling

Ver archivo separado: [Exception-Handling.md](Exception-Handling.md)

## Mejores Prácticas

### 1. No crear GlobalScope

```kotlin
// ❌ MALO
GlobalScope.launch {
    // Puede causar leaks
}

// ✅ BUENO
viewModelScope.launch {
    // Se cancela con el ViewModel
}
```

### 2. Usar Dispatchers Apropiados

```kotlin
// ❌ MALO - operación IO en Main
viewModelScope.launch {
    database.query() // Bloquea UI
}

// ✅ BUENO
viewModelScope.launch {
    withContext(Dispatchers.IO) {
        database.query()
    }
}
```

### 3. Manejar Cancelación

```kotlin
// ✅ BUENO - respetar cancelación
suspend fun doWork() {
    repeat(100) { i ->
        ensureActive() // Verificar cancelación
        processItem(i)
    }
}
```

## Recursos

- [Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html)
- [Android Coroutines](https://developer.android.com/kotlin/coroutines)
- [Coroutines Best Practices](https://developer.android.com/kotlin/coroutines/coroutines-best-practices)
- [Testing Coroutines](https://developer.android.com/kotlin/coroutines/test)

---
*Las coroutines son fundamentales para programación asíncrona moderna en Android*
