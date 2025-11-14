# Recomposition en Jetpack Compose

## ¿Qué es la Recomposition?

La **recomposition** es el proceso por el cual Compose vuelve a ejecutar composables cuando sus datos cambian. Es fundamental para construir UIs reactivas y eficientes.

## Conceptos Clave

### 1. Cuándo Ocurre la Recomposition

```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    
    // Esta función se recompone cuando 'count' cambia
    Button(onClick = { count++ }) {
        Text("Clicks: $count")
    }
}
```

**Triggers de Recomposition:**
- Cambio en `State<T>` (mutableStateOf)
- Cambio en `Flow` observado con collectAsState()
- Cambio en `LiveData` observado con observeAsState()
- Cambio en parámetros del composable

### 2. Scope de Recomposition

```kotlin
@Composable
fun SmartRecomposition() {
    var name by remember { mutableStateOf("Carlos") }
    var age by remember { mutableStateOf(25) }
    
    Column {
        // Solo este Text se recompone cuando 'name' cambia
        Text("Name: $name")
        
        // Solo este Text se recompone cuando 'age' cambia
        Text("Age: $age")
        
        // Este nunca se recompone (no depende de estados)
        Text("Fixed text")
        
        Button(onClick = { name = "Ana" }) {
            Text("Change Name")
        }
    }
}
```

**Principio:** Compose es inteligente y solo recompone las partes que realmente cambiaron.

## Optimización de Recomposition

### 1. Estabilidad de Parámetros

```kotlin
// ❌ MALO - Se recompone innecesariamente
data class User(var name: String, var age: Int)

@Composable
fun UserCard(user: User) {
    Text("${user.name}, ${user.age}")
}

// ✅ BUENO - Tipo estable
@Immutable
data class User(val name: String, val age: Int)

@Composable
fun UserCard(user: User) {
    Text("${user.name}, ${user.age}")
}

// ✅ MEJOR - Parámetros primitivos
@Composable
fun UserCard(name: String, age: Int) {
    Text("$name, $age")
}
```

### 2. Derivar Estado Correctamente

```kotlin
@Composable
fun SearchScreen() {
    var query by remember { mutableStateOf("") }
    
    // ❌ MALO - Crea un nuevo objeto en cada recomposition
    val results = searchResults(query)
    
    // ✅ BUENO - Solo recalcula cuando query cambia
    val results by remember(query) {
        derivedStateOf { searchResults(query) }
    }
    
    // ✅ MEJOR - Para operaciones pesadas
    val results by produceState(initialValue = emptyList(), query) {
        value = withContext(Dispatchers.IO) {
            performHeavySearch(query)
        }
    }
}
```

### 3. Remember con Keys

```kotlin
@Composable
fun ItemsList(items: List<Item>) {
    LazyColumn {
        items(items) { item ->
            // ❌ MALO - remember sin key
            val expanded = remember { mutableStateOf(false) }
            
            // ✅ BUENO - remember con key única
            val expanded = remember(item.id) { mutableStateOf(false) }
            
            ItemCard(item, expanded)
        }
    }
}
```

### 4. Lambdas Estables

```kotlin
@Composable
fun ParentComposable() {
    var count by remember { mutableStateOf(0) }
    
    // ❌ MALO - Nueva lambda en cada recomposition
    ChildComposable(onClick = { count++ })
    
    // ✅ BUENO - Lambda estable
    val onClickStable = remember { { count++ } }
    ChildComposable(onClick = onClickStable)
    
    // ✅ MEJOR - Si solo necesitas actualizar estado
    ChildComposable(onClick = remember { { count++ } })
}

@Composable
fun ChildComposable(onClick: () -> Unit) {
    Button(onClick = onClick) {
        Text("Click")
    }
}
```

## Técnicas Avanzadas de Optimización

### 1. @Stable y @Immutable

```kotlin
// Indica a Compose que este tipo es estable
@Stable
class MutableCounter(initial: Int) {
    var value by mutableStateOf(initial)
        private set
    
    fun increment() {
        value++
    }
}

// Indica que este tipo nunca cambia después de construcción
@Immutable
data class UserProfile(
    val id: String,
    val name: String,
    val email: String
)

@Composable
fun ProfileCard(profile: UserProfile) {
    // Solo se recompone si la referencia de profile cambia
    // No se recompone si cambia contenido mutable
    Text(profile.name)
}
```

### 2. SnapshotStateList y SnapshotStateMap

```kotlin
@Composable
fun TodoList() {
    // ✅ Lista observable que trigger recomposition solo cuando cambia
    val todos = remember { mutableStateListOf<Todo>() }
    
    LazyColumn {
        items(todos) { todo ->
            TodoItem(todo)
        }
    }
    
    Button(onClick = { todos.add(Todo("New task")) }) {
        Text("Add Todo")
    }
}
```

### 3. Recomposition Scope con movableContentOf

```kotlin
@Composable
fun MovableContentExample() {
    var showInColumn by remember { mutableStateOf(true) }
    
    // El contenido se mantiene sin recomponerse completamente
    val content = remember {
        movableContentOf {
            ExpensiveComposable()
        }
    }
    
    if (showInColumn) {
        Column {
            content()
        }
    } else {
        Row {
            content()
        }
    }
}
```

### 4. Evitar Recomposition con key

```kotlin
@Composable
fun ConditionalContent(showVersionA: Boolean) {
    // Sin key, cambia entre versiones recomponiendo
    if (showVersionA) {
        ContentVersionA()
    } else {
        ContentVersionB()
    }
    
    // ✅ Con key, mantiene el estado separado
    key(showVersionA) {
        if (showVersionA) {
            ContentVersionA()
        } else {
            ContentVersionB()
        }
    }
}
```

## Debugging de Recomposition

### 1. Compose Compiler Metrics

```kotlin
// En build.gradle
android {
    kotlinOptions {
        freeCompilerArgs += [
            "-P",
            "plugin:androidx.compose.compiler.plugins.kotlin:metricsDestination=" + 
                project.buildDir.absolutePath + "/compose_metrics",
            "-P",
            "plugin:androidx.compose.compiler.plugins.kotlin:reportsDestination=" + 
                project.buildDir.absolutePath + "/compose_reports"
        ]
    }
}
```

### 2. Recomposition Counter

```kotlin
@Composable
fun RecompositionCounter() {
    val counter = remember { mutableStateOf(0) }
    
    SideEffect {
        counter.value++
    }
    
    Text("Recompositions: ${counter.value}")
}
```

### 3. Layout Inspector

```kotlin
// Usar Android Studio Layout Inspector para:
// - Ver cuántas veces se recompone cada composable
// - Identificar recomposiciones innecesarias
// - Analizar la jerarquía de composables
```

## Patrones Comunes de Optimización

### 1. Separar Estado de Presentación

```kotlin
// ❌ MALO - Todo se recompone cuando cambia cualquier estado
@Composable
fun UserProfile() {
    var name by remember { mutableStateOf("") }
    var bio by remember { mutableStateOf("") }
    var avatar by remember { mutableStateOf("") }
    
    Column {
        Image(avatar)
        TextField(name, onValueChange = { name = it })
        TextField(bio, onValueChange = { bio = it })
    }
}

// ✅ BUENO - Separar en composables más pequeños
@Composable
fun UserProfile() {
    Column {
        AvatarSection()
        NameSection()
        BioSection()
    }
}

@Composable
private fun NameSection() {
    var name by remember { mutableStateOf("") }
    TextField(name, onValueChange = { name = it })
}
```

### 2. Estado Derivado con derivedStateOf

```kotlin
@Composable
fun FilteredList(items: List<Item>) {
    var searchQuery by remember { mutableStateOf("") }
    
    // ✅ Solo recalcula cuando searchQuery o items cambian
    val filteredItems by remember(searchQuery, items) {
        derivedStateOf {
            items.filter { it.name.contains(searchQuery, ignoreCase = true) }
        }
    }
    
    Column {
        SearchBar(searchQuery) { searchQuery = it }
        LazyColumn {
            items(filteredItems) { item ->
                ItemCard(item)
            }
        }
    }
}
```

### 3. Defer Reads para Lazy Layouts

```kotlin
@Composable
fun LazyItemsList(items: List<Item>) {
    LazyColumn {
        items(
            items = items,
            key = { it.id }
        ) { item ->
            // ✅ Solo se recompone este item cuando sus datos cambian
            ItemCard(item)
        }
    }
}
```

## Anti-Patrones a Evitar

### 1. Crear Objetos en Recomposition

```kotlin
// ❌ MALO - Nuevo objeto en cada recomposition
@Composable
fun BadExample() {
    val viewModel = MyViewModel() // ¡Nuevo ViewModel cada vez!
    // ...
}

// ✅ BUENO
@Composable
fun GoodExample() {
    val viewModel: MyViewModel = viewModel()
    // ...
}
```

### 2. Modificar Estado Durante Composition

```kotlin
// ❌ MALO - Modifica estado durante composition
@Composable
fun BadExample() {
    var counter by remember { mutableStateOf(0) }
    counter++ // ¡Causa loop infinito!
    
    Text("Counter: $counter")
}

// ✅ BUENO - Modifica en event handler
@Composable
fun GoodExample() {
    var counter by remember { mutableStateOf(0) }
    
    Column {
        Text("Counter: $counter")
        Button(onClick = { counter++ }) {
            Text("Increment")
        }
    }
}
```

### 3. Usar var en lugar de State

```kotlin
// ❌ MALO - No trigger recomposition
@Composable
fun BadExample() {
    var name = "Carlos" // Variable regular
    
    Button(onClick = { name = "Ana" }) { // ¡No se actualiza la UI!
        Text(name)
    }
}

// ✅ BUENO
@Composable
fun GoodExample() {
    var name by remember { mutableStateOf("Carlos") }
    
    Button(onClick = { name = "Ana" }) {
        Text(name)
    }
}
```

## Checklist de Optimización

- [ ] Usar tipos inmutables o @Stable/@Immutable
- [ ] Remember lambdas y objetos cuando sea apropiado
- [ ] Usar derivedStateOf para cálculos derivados
- [ ] Separar composables en piezas más pequeñas
- [ ] Usar keys en LazyLists
- [ ] Evitar crear objetos durante composition
- [ ] Usar Compose Compiler Metrics para analizar
- [ ] Profiler para medir recompositions reales
- [ ] State hoisting apropiado
- [ ] Evitar lecturas de estado innecesarias

## Herramientas de Análisis

### Compose Compiler Reports

```bash
# Generar reports
./gradlew assembleRelease -P \
  plugin:androidx.compose.compiler.plugins.kotlin:metricsDestination=./metrics \
  plugin:androidx.compose.compiler.plugins.kotlin:reportsDestination=./reports

# Archivos generados:
# - *-composables.txt: Lista de composables y su estabilidad
# - *-classes.txt: Estabilidad de clases
# - *-composables.csv: Métricas de composables
```

### Layout Inspector (Android Studio)

1. Tools → Layout Inspector → Select Process
2. Activar "Show Recomposition Counts"
3. Observar composables que se recomponen frecuentemente
4. Investigar y optimizar los hotspots

## Recursos

- [Compose Performance](https://developer.android.com/jetpack/compose/performance)
- [Thinking in Compose](https://developer.android.com/jetpack/compose/mental-model)
- [Compose Stability](https://developer.android.com/jetpack/compose/performance/stability)
- [Compose Compiler Metrics](https://github.com/androidx/androidx/blob/androidx-main/compose/compiler/design/compiler-metrics.md)

---
*La optimización de recomposition es clave para apps performantes en Compose*
