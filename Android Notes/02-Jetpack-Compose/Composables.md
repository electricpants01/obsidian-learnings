# Composables - Funciones Declarativas de UI

## Descripción

Los Composables son funciones anotadas con `@Composable` que definen elementos de UI de manera declarativa en Jetpack Compose. Permiten crear interfaces de usuario modernas y reactivas sin XML.

### Concepto Principal

**UI como función del estado**

En Compose, la UI es resultado directo del estado actual. Cuando el estado cambia, Compose automáticamente actualiza (recompone) la UI.

### Ventajas

- Código más conciso y legible
- Menos boilerplate que Views XML
- Reutilización fácil de componentes
- Preview en tiempo real
- Type-safe
- Integración perfecta con Kotlin

## Ejemplo Básico

```kotlin
@Composable
fun Greeting(name: String) {
    Text(text = "Hello, $name!")
}

// Preview
@Preview(showBackground = true)
@Composable
fun GreetingPreview() {
    Greeting(name = "Android")
}
```

## Composables con Estado

```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = "Count: $count",
            style = MaterialTheme.typography.headlineLarge
        )
        
        Spacer(modifier = Modifier.height(16.dp))
        
        Button(onClick = { count++ }) {
            Text("Increment")
        }
    }
}
```

## Composables Reutilizables

```kotlin
// Componente reutilizable
@Composable
fun UserCard(
    user: User,
    onClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    Card(
        modifier = modifier
            .fillMaxWidth()
            .clickable(onClick = onClick),
        elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
    ) {
        Row(
            modifier = Modifier.padding(16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            AsyncImage(
                model = user.avatar,
                contentDescription = "Avatar",
                modifier = Modifier
                    .size(48.dp)
                    .clip(CircleShape)
            )
            
            Spacer(modifier = Modifier.width(16.dp))
            
            Column {
                Text(
                    text = user.name,
                    style = MaterialTheme.typography.titleMedium
                )
                Text(
                    text = user.email,
                    style = MaterialTheme.typography.bodySmall,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
        }
    }
}

// Uso
@Composable
fun UserList(users: List<User>) {
    LazyColumn {
        items(users) { user ->
            UserCard(
                user = user,
                onClick = { /* Handle click */ },
                modifier = Modifier.padding(horizontal = 16.dp, vertical = 8.dp)
            )
        }
    }
}
```

## Composables con Parámetros Opcionales

```kotlin
@Composable
fun CustomButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
    colors: ButtonColors = ButtonDefaults.buttonColors(),
    icon: (@Composable () -> Unit)? = null
) {
    Button(
        onClick = onClick,
        enabled = enabled,
        colors = colors,
        modifier = modifier
    ) {
        icon?.invoke()
        if (icon != null) {
            Spacer(modifier = Modifier.width(8.dp))
        }
        Text(text)
    }
}

// Usos variados
CustomButton(
    text = "Click me",
    onClick = { }
)

CustomButton(
    text = "Save",
    onClick = { },
    icon = { Icon(Icons.Default.Save, contentDescription = null) }
)
```

## Documentación Oficial

- [Thinking in Compose](https://developer.android.com/jetpack/compose/mental-model)
- [Composable functions](https://developer.android.com/jetpack/compose/composables)
- [Compose Tutorial](https://developer.android.com/jetpack/compose/tutorial)
- [Compose Basics](https://developer.android.com/jetpack/compose/layouts/basics)

## Videos Recomendados

- [Jetpack Compose Basics](https://www.youtube.com/watch?v=6_wK_Ud8--0) - Android Developers
- [Thinking in Compose](https://www.youtube.com/watch?v=SMOhl9RK0BA) - Google I/O
- [Compose for Beginners](https://www.youtube.com/watch?v=6BRlI5zfCCk) - Philipp Lackner

## Mejores Prácticas

1. **Composables Pequeños**: Crear funciones pequeñas y reutilizables
2. **Nombres Descriptivos**: Usar PascalCase y nombres que describan el contenido
3. **Modifier al final**: Pasar Modifier como último parámetro con valor por defecto
4. **Hoisting de estado**: Elevar el estado cuando sea compartido
5. **Preview todo**: Crear previews para facilitar el desarrollo
6. **Evitar side-effects**: Los composables deben ser funciones puras

## Lifecycle de Composables

```kotlin
@Composable
fun LifecycleExample() {
    // 1. Composition: Se ejecuta la primera vez
    println("Composing...")
    
    // 2. Recomposition: Se ejecuta cuando cambia el estado
    DisposableEffect(Unit) {
        println("Entered composition")
        
        // 3. Leaving composition
        onDispose {
            println("Leaving composition")
        }
    }
}
```

## Composables vs Views

| Aspecto | Composables | Views (XML) |
|---------|-------------|-------------|
| Declaración | Funciones Kotlin | XML + Kotlin |
| Reactividad | Automática | Manual (findViewById, listeners) |
| Reutilización | Muy fácil | Más complejo |
| Preview | Tiempo real | Limitado |
| Type Safety | Sí | Parcial |
| Aprendizaje | Curva moderna | Tradicional |

## Errores Comunes

❌ **Llamar funciones suspendidas directamente**
```kotlin
@Composable
fun BadExample() {
    // NO hacer esto
    val data = repository.getData() // Error: suspend function
}
```

✅ **Usar LaunchedEffect para código suspendido**
```kotlin
@Composable
fun GoodExample() {
    var data by remember { mutableStateOf<Data?>(null) }
    
    LaunchedEffect(Unit) {
        data = repository.getData()
    }
    
    data?.let { DisplayData(it) }
}
```

❌ **Modificar estado directamente en composición**
```kotlin
@Composable
fun BadCounter() {
    var count = 0  // Se pierde en recomposition
    
    Button(onClick = { count++ }) {  // No funciona
        Text("Count: $count")
    }
}
```

✅ **Usar remember para estado**
```kotlin
@Composable
fun GoodCounter() {
    var count by remember { mutableStateOf(0) }
    
    Button(onClick = { count++ }) {
        Text("Count: $count")
    }
}
```

## Preview Avanzado

```kotlin
// Preview con diferentes configuraciones
@Preview(name = "Light Mode", showBackground = true)
@Preview(
    name = "Dark Mode",
    uiMode = Configuration.UI_MODE_NIGHT_YES,
    showBackground = true
)
@Preview(name = "Large Font", fontScale = 1.5f)
@Composable
fun UserCardPreviews() {
    MaterialTheme {
        UserCard(
            user = User(
                name = "John Doe",
                email = "john@example.com",
                avatar = ""
            ),
            onClick = {}
        )
    }
}
```

## Recursos Adicionales

- [Compose Samples](https://github.com/android/compose-samples)
- [Compose Pathway](https://developer.android.com/courses/pathways/compose)
- [Jetpack Compose Codelabs](https://developer.android.com/codelabs/jetpack-compose-basics)
- [Compose Material 3](https://developer.android.com/jetpack/compose/designsystems/material3)

## Tags
#compose #jetpack #ui #declarative #android

---
*Última actualización: 2025-01-14*
