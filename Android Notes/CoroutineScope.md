# CoroutineScope

## ¿Qué es CoroutineScope?

**CoroutineScope** define el alcance de vida de las coroutines. Cuando se cancela un scope, todas sus coroutines hijas se cancelan automáticamente.

## Scopes en Android

### 1. ViewModelScope

```kotlin
class MyViewModel : ViewModel() {
    fun loadData() {
        viewModelScope.launch {
            // Se cancela cuando ViewModel se limpia
            val data = repository.getData()
        }
    }
}

// Dependency
implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0")
```

### 2. LifecycleScope

```kotlin
class MyFragment : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        // lifecycleScope - se cancela con el lifecycle
        lifecycleScope.launch {
            // Work
        }
        
        // viewLifecycleOwner.lifecycleScope - se cancela con la vista
        viewLifecycleOwner.lifecycleScope.launch {
            viewModel.uiState.collect { state ->
                updateUI(state)
            }
        }
    }
}
```

### 3. Scope Personalizado

```kotlin
class MyRepository {
    private val scope = CoroutineScope(
        Dispatchers.Default + SupervisorJob()
    )
    
    fun loadData() {
        scope.launch {
            // Work
        }
    }
    
    fun cleanup() {
        scope.cancel()
    }
}
```

## Structured Concurrency

```kotlin
// Parent-child relationship
viewModelScope.launch { // Parent
    launch { // Child 1
        delay(1000)
    }
    launch { // Child 2
        delay(2000)
    }
    // Espera a ambos hijos
}
```

## Job Hierarchy

```kotlin
val parentJob = Job()
val scope = CoroutineScope(parentJob)

val childJob = scope.launch {
    // Work
}

parentJob.cancel() // Cancela todos los hijos
```

## SupervisorJob

```kotlin
// SupervisorJob - un hijo puede fallar sin cancelar otros
val supervisorScope = CoroutineScope(SupervisorJob())

supervisorScope.launch {
    throw Exception() // No cancela otros hijos
}

supervisorScope.launch {
    // Continúa ejecutándose
}
```

## Recursos

- [CoroutineScope](https://kotlinlang.org/docs/coroutine-context-and-dispatchers.html)
- [Android Scopes](https://developer.android.com/topic/libraries/architecture/coroutines)
