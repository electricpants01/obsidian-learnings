# State Management en Compose

## Descripción

State Management en Jetpack Compose se refiere a cómo se maneja y almacena el estado de la UI. El estado es cualquier valor que puede cambiar con el tiempo y que afecta lo que se muestra en la pantalla.

### Concepto Principal

**UI = f(state)**

La UI es una función pura del estado. Cuando el estado cambia, Compose recompone automáticamente las partes afectadas de la UI.

### APIs Principales

1. **remember**: Preserva estado durante recomposiciones
2. **mutableStateOf**: Crea estado observable
3. **derivedStateOf**: Estado derivado de otros estados
4. **rememberSaveable**: Sobrevive a cambios de configuración

## remember y mutableStateOf

```kotlin
@Composable
fun Counter() {
    // remember preserva el valor entre recomposiciones
    var count by remember { mutableStateOf(0) }
    
    Column {
        Text("Count: $count")
        Button(onClick = { count++ }) {
            Text("Increment")
        }
    }
}

// Sin remember - INCORRECTO
@Composable
fun BadCounter() {
    var count by mutableStateOf(0) // Se reinicia en cada recomposición
    
    Button(onClick = { count++ }) {
        Text("Count: $count") // Siempre muestra 0
    }
}
```

## rememberSaveable

```kotlin
@Composable
fun SearchScreen() {
    // Sobrevive a cambios de configuración (rotación)
    var searchQuery by rememberSaveable { mutableStateOf("") }
    
    TextField(
        value = searchQuery,
        onValueChange = { searchQuery = it },
        label = { Text("Search") }
    )
}

// Con custom Saver
data class User(val name: String, val age: Int)

@Composable
fun UserEditor() {
    var user by rememberSaveable(
        stateSaver = Saver(
            save = { listOf(it.name, it.age) },
            restore = { User(it[0] as String, it[1] as Int) }
        )
    ) {
        mutableStateOf(User("", 0))
    }
    
    Column {
        TextField(
            value = user.name,
            onValueChange = { user = user.copy(name = it) }
        )
        TextField(
            value = user.age.toString(),
            onValueChange = { user = user.copy(age = it.toIntOrNull() ?: 0) }
        )
    }
}
```

## derivedStateOf

```kotlin
@Composable
fun ShoppingCart(items: List<Item>) {
    // Se recalcula solo cuando items cambia
    val totalPrice by remember(items) {
        derivedStateOf {
            items.sumOf { it.price }
        }
    }
    
    val itemCount by remember(items) {
        derivedStateOf {
            items.size
        }
    }
    
    Column {
        Text("Items: $itemCount")
        Text("Total: $$totalPrice")
        
        LazyColumn {
            items(items) { item ->
                ItemRow(item)
            }
        }
    }
}

// Uso avanzado de derivedStateOf
@Composable
fun MessageList(messages: List<Message>) {
    val listState = rememberLazyListState()
    
    // Solo se recalcula cuando el índice cambia
    val showScrollToTop by remember {
        derivedStateOf {
            listState.firstVisibleItemIndex > 0
        }
    }
    
    Box {
        LazyColumn(state = listState) {
            items(messages) { message ->
                MessageItem(message)
            }
        }
        
        if (showScrollToTop) {
            FloatingActionButton(
                onClick = { /* scroll to top */ }
            ) {
                Icon(Icons.Default.ArrowUpward, "Scroll to top")
            }
        }
    }
}
```

## State Hoisting

```kotlin
// ❌ Estado no hoisted - menos reutilizable
@Composable
fun SearchField() {
    var query by remember { mutableStateOf("") }
    
    TextField(
        value = query,
        onValueChange = { query = it }
    )
}

// ✅ Estado hoisted - más reutilizable
@Composable
fun SearchField(
    query: String,
    onQueryChange: (String) -> Unit,
    modifier: Modifier = Modifier
) {
    TextField(
        value = query,
        onValueChange = onQueryChange,
        modifier = modifier
    )
}

// Uso con estado hoisted
@Composable
fun SearchScreen() {
    var query by remember { mutableStateOf("") }
    
    Column {
        SearchField(
            query = query,
            onQueryChange = { query = it }
        )
        
        SearchResults(query = query)
    }
}
```

## State con ViewModel

```kotlin
class UserViewModel : ViewModel() {
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users: StateFlow<List<User>> = _users.asStateFlow()
    
    private val _isLoading = MutableStateFlow(false)
    val isLoading: StateFlow<Boolean> = _isLoading.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _isLoading.value = true
            try {
                _users.value = repository.getUsers()
            } finally {
                _isLoading.value = false
            }
        }
    }
}

@Composable
fun UserListScreen(viewModel: UserViewModel = hiltViewModel()) {
    val users by viewModel.users.collectAsState()
    val isLoading by viewModel.isLoading.collectAsState()
    
    LaunchedEffect(Unit) {
        viewModel.loadUsers()
    }
    
    when {
        isLoading -> LoadingIndicator()
        users.isEmpty() -> EmptyState()
        else -> UserList(users)
    }
}
```

## Documentación Oficial

- [State and Jetpack Compose](https://developer.android.com/jetpack/compose/state)
- [State hoisting](https://developer.android.com/jetpack/compose/state-hoisting)
- [Where to hoist state](https://developer.android.com/jetpack/compose/state#where-to-hoist)
- [Save UI state](https://developer.android.com/jetpack/compose/state-saving)

## Videos Recomendados

- [State in Jetpack Compose](https://www.youtube.com/watch?v=mymWGMy9pYI) - Android Developers
- [State Management Best Practices](https://www.youtube.com/watch?v=rmv2ug-wW4U) - Google I/O
- [Compose State Deep Dive](https://www.youtube.com/watch?v=PMMY23F0CFg) - Philipp Lackner

## Mejores Prácticas

1. **Hoist cuando sea compartido**: Eleva el estado al mínimo ancestro común
2. **remember para valores calculados**: Evita cálculos innecesarios
3. **rememberSaveable para datos importantes**: Datos que deben sobrevivir rotaciones
4. **derivedStateOf para estados derivados**: Optimiza cálculos basados en otros estados
5. **StateFlow en ViewModel**: Para estado que sobrevive al ciclo de vida de Compose

## Patterns de State Management

### Unidirectional Data Flow (UDF)

```kotlin
// State holder
data class TodoState(
    val todos: List<Todo> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)

// Events
sealed class TodoEvent {
    data class AddTodo(val title: String) : TodoEvent()
    data class ToggleTodo(val id: String) : TodoEvent()
    data class DeleteTodo(val id: String) : TodoEvent()
}

// ViewModel con UDF
class TodoViewModel : ViewModel() {
    private val _state = MutableStateFlow(TodoState())
    val state: StateFlow<TodoState> = _state.asStateFlow()
    
    fun onEvent(event: TodoEvent) {
        when (event) {
            is TodoEvent.AddTodo -> addTodo(event.title)
            is TodoEvent.ToggleTodo -> toggleTodo(event.id)
            is TodoEvent.DeleteTodo -> deleteTodo(event.id)
        }
    }
    
    private fun addTodo(title: String) {
        val newTodo = Todo(id = UUID.randomUUID().toString(), title = title)
        _state.update { it.copy(todos = it.todos + newTodo) }
    }
}

// UI
@Composable
fun TodoScreen(viewModel: TodoViewModel = hiltViewModel()) {
    val state by viewModel.state.collectAsState()
    
    TodoContent(
        state = state,
        onEvent = viewModel::onEvent
    )
}
```

## Errores Comunes

❌ **Modificar lista mutable directamente**
```kotlin
@Composable
fun BadList() {
    val items = remember { mutableListOf<String>() }
    
    Button(onClick = {
        items.add("New Item") // No recompone!
    }) {
        Text("Add")
    }
}
```

✅ **Usar mutableStateOf con lista inmutable**
```kotlin
@Composable
fun GoodList() {
    var items by remember { mutableStateOf(listOf<String>()) }
    
    Button(onClick = {
        items = items + "New Item" // Recompone correctamente
    }) {
        Text("Add")
    }
}
```

❌ **Estado sin remember**
```kotlin
@Composable
fun BadCounter() {
    var count = 0 // Se pierde en recomposición
    
    Button(onClick = { count++ }) {
        Text("$count")
    }
}
```

✅ **Estado con remember**
```kotlin
@Composable
fun GoodCounter() {
    var count by remember { mutableStateOf(0) }
    
    Button(onClick = { count++ }) {
        Text("$count")
    }
}
```

## Recursos Adicionales

- [State Codelab](https://developer.android.com/codelabs/jetpack-compose-state)
- [Advanced State](https://developer.android.com/jetpack/compose/state-advanced)
- [ViewModel in Compose](https://developer.android.com/jetpack/compose/libraries#viewmodel)

## Tags
#compose #state #statemanagement #remember #mutablestateof

---
*Última actualización: 2025-01-14*
