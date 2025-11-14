# Dispatchers

## Tipos de Dispatchers

### 1. Dispatchers.Main
```kotlin
// UI updates
viewModelScope.launch(Dispatchers.Main) {
    updateUI()
}
```

### 2. Dispatchers.IO
```kotlin
// Network, DB, File I/O
withContext(Dispatchers.IO) {
    val data = api.fetchData()
    database.save(data)
}
```

### 3. Dispatchers.Default
```kotlin
// CPU-intensive work
withContext(Dispatchers.Default) {
    val result = processLargeData()
}
```

### 4. Dispatchers.Unconfined
```kotlin
// No recomendado - usa dispatcher del caller
launch(Dispatchers.Unconfined) {
    // Avoid using this
}
```

## Dispatcher Personalizado

```kotlin
val customDispatcher = Executors.newFixedThreadPool(4).asCoroutineDispatcher()

launch(customDispatcher) {
    // Work on custom pool
}

// Cleanup
customDispatcher.close()
```

## limitedParallelism

```kotlin
val limitedDispatcher = Dispatchers.IO.limitedParallelism(5)

repeat(10) {
    launch(limitedDispatcher) {
        // Max 5 en paralelo
    }
}
```

## Testing

```kotlin
@Test
fun test() = runTest {
    val testDispatcher = UnconfinedTestDispatcher()
    
    withContext(testDispatcher) {
        // Test
    }
}
```

## Recursos
- [Dispatchers Guide](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html)
