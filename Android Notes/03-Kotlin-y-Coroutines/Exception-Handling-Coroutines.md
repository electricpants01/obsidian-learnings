# Exception Handling en Coroutines

## Try-Catch

```kotlin
viewModelScope.launch {
    try {
        val data = repository.getData()
        _state.value = Success(data)
    } catch (e: Exception) {
        _state.value = Error(e.message)
    }
}
```

## CoroutineExceptionHandler

```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    Log.e("Coroutine", "Error: $exception")
}

viewModelScope.launch(handler) {
    throw Exception("Error!")
}
```

## SupervisorJob

```kotlin
val supervisorJob = SupervisorJob()
val scope = CoroutineScope(supervisorJob)

scope.launch {
    throw Exception() // No cancela otras coroutines
}
```

## Async Exception

```kotlin
coroutineScope {
    val deferred = async {
        throw Exception()
    }
    
    try {
        deferred.await()
    } catch (e: Exception) {
        // Handle
    }
}
```

## Recursos
- [Exception Handling](https://kotlinlang.org/docs/exception-handling.html)
