# Structured Concurrency

## Principio

Las coroutines hijas no pueden sobrevivir a su padre.

## Jerarquía

```kotlin
viewModelScope.launch { // Parent
    launch { task1() } // Child 1
    launch { task2() } // Child 2
    // Parent espera a todos los hijos
}
```

## coroutineScope vs supervisorScope

```kotlin
// coroutineScope - un fallo cancela todos
suspend fun loadData() = coroutineScope {
    launch { loadUsers() }
    launch { throw Exception() } // Cancela loadUsers
}

// supervisorScope - fallos independientes
suspend fun loadDataSafe() = supervisorScope {
    launch { loadUsers() }
    launch { throw Exception() } // NO cancela loadUsers
}
```

## Cancelación en Cascada

```kotlin
val parent = launch {
    val child = launch {
        delay(Long.MAX_VALUE)
    }
}

parent.cancel() // Cancela child también
```

## Recursos
- [Structured Concurrency](https://kotlinlang.org/docs/coroutines-basics.html#structured-concurrency)
