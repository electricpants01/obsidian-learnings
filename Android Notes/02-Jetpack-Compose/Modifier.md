# Modifier en Jetpack Compose

## ¿Qué es un Modifier?

Un **Modifier** es un objeto que permite modificar o decorar composables en Jetpack Compose. Se utiliza para definir el diseño, comportamiento, apariencia e interacción de los elementos de UI.

## Conceptos Fundamentales

### 1. Cadena de Modificadores

```kotlin
@Composable
fun ModifierChainExample() {
    Box(
        modifier = Modifier
            .size(100.dp)           // 1. Define tamaño
            .background(Color.Blue)  // 2. Fondo azul
            .padding(16.dp)          // 3. Padding interno
            .border(2.dp, Color.Red) // 4. Borde rojo
    )
}
```

**El orden importa:**
```kotlin
// Diferentes resultados según el orden
Box(
    Modifier
        .padding(16.dp)          // Padding primero
        .background(Color.Blue)  // Luego fondo
) // El padding NO tiene color de fondo

Box(
    Modifier
        .background(Color.Blue)  // Fondo primero
        .padding(16.dp)          // Luego padding
) // El padding TIENE color de fondo
```

### 2. Modifier como Parámetro

```kotlin
// ✅ BUENA PRÁCTICA: Siempre aceptar Modifier como primer parámetro
@Composable
fun MyCard(
    modifier: Modifier = Modifier,  // Primer parámetro, valor default
    title: String,
    content: String
) {
    Card(
        modifier = modifier  // Aplicar modifier del padre
            .fillMaxWidth()
            .padding(8.dp)
    ) {
        Column {
            Text(title)
            Text(content)
        }
    }
}

// Uso
MyCard(
    modifier = Modifier.padding(16.dp),
    title = "Título",
    content = "Contenido"
)
```

## Categorías de Modificadores

### 1. Tamaño (Size)

```kotlin
@Composable
fun SizeModifiers() {
    Column {
        // Tamaño fijo
        Box(Modifier.size(100.dp))
        Box(Modifier.width(100.dp).height(50.dp))
        
        // Tamaño relativo
        Box(Modifier.fillMaxWidth())        // 100% ancho
        Box(Modifier.fillMaxHeight())       // 100% alto
        Box(Modifier.fillMaxSize())         // 100% ambos
        
        // Tamaño fraccionado
        Box(Modifier.fillMaxWidth(0.5f))    // 50% ancho
        
        // Tamaño mínimo/máximo
        Box(
            Modifier
                .widthIn(min = 100.dp, max = 300.dp)
                .heightIn(min = 50.dp)
        )
        
        // Tamaño por defecto (si no hay constraints)
        Box(Modifier.defaultMinSize(minWidth = 100.dp))
        
        // Aspect ratio
        Box(Modifier.aspectRatio(16f / 9f))
    }
}
```

### 2. Padding y Spacing

```kotlin
@Composable
fun PaddingModifiers() {
    Column {
        // Padding uniforme
        Box(Modifier.padding(16.dp))
        
        // Padding horizontal y vertical
        Box(Modifier.padding(horizontal = 16.dp, vertical = 8.dp))
        
        // Padding específico por lado
        Box(
            Modifier.padding(
                start = 16.dp,
                top = 8.dp,
                end = 16.dp,
                bottom = 8.dp
            )
        )
        
        // Padding absoluto (ignorando RTL)
        Box(
            Modifier.absolutePadding(
                left = 16.dp,
                right = 16.dp
            )
        )
    }
}
```

### 3. Offset

```kotlin
@Composable
fun OffsetModifiers() {
    Box {
        // Offset fijo
        Text(
            "Moved",
            Modifier.offset(x = 10.dp, y = 20.dp)
        )
        
        // Offset absoluto (ignorando RTL)
        Text(
            "Absolute",
            Modifier.absoluteOffset(x = 10.dp, y = 20.dp)
        )
        
        // Offset con lambda (para animaciones)
        var offsetX by remember { mutableStateOf(0.dp) }
        Text(
            "Dynamic",
            Modifier.offset { IntOffset(offsetX.roundToPx(), 0) }
        )
    }
}
```

### 4. Decoración Visual

```kotlin
@Composable
fun VisualModifiers() {
    Column {
        // Background
        Box(Modifier.background(Color.Blue))
        Box(
            Modifier.background(
                brush = Brush.horizontalGradient(
                    colors = listOf(Color.Red, Color.Blue)
                )
            )
        )
        
        // Border
        Box(
            Modifier.border(
                width = 2.dp,
                color = Color.Red,
                shape = RoundedCornerShape(8.dp)
            )
        )
        
        // Shadow
        Box(Modifier.shadow(elevation = 8.dp, shape = RoundedCornerShape(8.dp)))
        
        // Alpha (transparencia)
        Box(Modifier.alpha(0.5f))
        
        // Clip
        Box(Modifier.clip(RoundedCornerShape(8.dp)))
        Box(Modifier.clip(CircleShape))
    }
}
```

### 5. Gestos e Interacción

```kotlin
@Composable
fun InteractionModifiers() {
    var clicks by remember { mutableStateOf(0) }
    
    Column {
        // Clickable
        Box(
            Modifier.clickable { clicks++ }
        ) {
            Text("Clicks: $clicks")
        }
        
        // Clickable con ripple personalizado
        Box(
            Modifier.clickable(
                indication = rememberRipple(color = Color.Red),
                interactionSource = remember { MutableInteractionSource() }
            ) {
                clicks++
            }
        )
        
        // Sin ripple
        Box(
            Modifier.clickable(
                indication = null,
                interactionSource = remember { MutableInteractionSource() }
            ) {
                clicks++
            }
        )
        
        // Combinable clickable
        Box(
            Modifier.combinedClickable(
                onClick = { /* Click simple */ },
                onLongClick = { /* Long click */ },
                onDoubleClick = { /* Double click */ }
            )
        )
        
        // Toggleable (para checkboxes, switches)
        Box(
            Modifier.toggleable(
                value = clicks % 2 == 0,
                onValueChange = { clicks++ }
            )
        )
        
        // Selectable (para radio buttons)
        Box(
            Modifier.selectable(
                selected = clicks > 5,
                onClick = { clicks++ }
            )
        )
    }
}
```

### 6. Scroll

```kotlin
@Composable
fun ScrollModifiers() {
    // Scroll vertical
    Column(
        Modifier
            .verticalScroll(rememberScrollState())
            .fillMaxSize()
    ) {
        repeat(50) { index ->
            Text("Item $index")
        }
    }
    
    // Scroll horizontal
    Row(
        Modifier.horizontalScroll(rememberScrollState())
    ) {
        repeat(20) { index ->
            Text("Item $index", Modifier.padding(16.dp))
        }
    }
    
    // Scroll con estado
    val scrollState = rememberScrollState()
    
    Column {
        Text("Scroll position: ${scrollState.value}")
        
        LaunchedEffect(Unit) {
            // Scroll programático
            scrollState.animateScrollTo(100)
        }
        
        Column(Modifier.verticalScroll(scrollState)) {
            // Content
        }
    }
}
```

### 7. Gestos Avanzados (Pointers)

```kotlin
@Composable
fun PointerModifiers() {
    var offset by remember { mutableStateOf(Offset.Zero) }
    
    Box(
        Modifier
            .size(200.dp)
            .background(Color.Blue)
            .pointerInput(Unit) {
                detectDragGestures { change, dragAmount ->
                    change.consume()
                    offset += dragAmount
                }
            }
            .offset { IntOffset(offset.x.roundToInt(), offset.y.roundToInt()) }
    )
    
    // Detectar taps
    Box(
        Modifier
            .size(100.dp)
            .pointerInput(Unit) {
                detectTapGestures(
                    onTap = { /* Tap simple */ },
                    onDoubleTap = { /* Doble tap */ },
                    onLongPress = { /* Long press */ },
                    onPress = { /* Press down */ }
                )
            }
    )
    
    // Transformaciones (scale, rotate, pan)
    var scale by remember { mutableStateOf(1f) }
    var rotation by remember { mutableStateOf(0f) }
    var offsetTransform by remember { mutableStateOf(Offset.Zero) }
    
    Box(
        Modifier
            .graphicsLayer(
                scaleX = scale,
                scaleY = scale,
                rotationZ = rotation,
                translationX = offsetTransform.x,
                translationY = offsetTransform.y
            )
            .pointerInput(Unit) {
                detectTransformGestures { _, pan, zoom, rotate ->
                    scale *= zoom
                    rotation += rotate
                    offsetTransform += pan
                }
            }
    )
}
```

### 8. Layout y Alignment

```kotlin
@Composable
fun LayoutModifiers() {
    Box {
        // Align dentro del padre
        Text(
            "Top Start",
            Modifier.align(Alignment.TopStart)
        )
        
        Text(
            "Center",
            Modifier.align(Alignment.Center)
        )
        
        Text(
            "Bottom End",
            Modifier.align(Alignment.BottomEnd)
        )
    }
    
    // Weight en Row/Column
    Row(Modifier.fillMaxWidth()) {
        Box(
            Modifier
                .weight(1f)
                .background(Color.Red)
        )
        Box(
            Modifier
                .weight(2f)
                .background(Color.Blue)
        )
    }
    
    // Layout personalizado
    Box(
        Modifier.layout { measurable, constraints ->
            val placeable = measurable.measure(constraints)
            layout(placeable.width, placeable.height) {
                placeable.place(0, 0)
            }
        }
    )
}
```

### 9. GraphicsLayer

```kotlin
@Composable
fun GraphicsLayerModifiers() {
    var rotation by remember { mutableStateOf(0f) }
    
    Box(
        Modifier
            .size(100.dp)
            .graphicsLayer {
                // Transformaciones
                rotationZ = rotation
                rotationX = 45f
                rotationY = 45f
                
                // Escala
                scaleX = 1.5f
                scaleY = 1.5f
                
                // Translación
                translationX = 100f
                translationY = 50f
                
                // Alpha y clipping
                alpha = 0.8f
                clip = true
                shape = RoundedCornerShape(8.dp)
                
                // Shadow
                shadowElevation = 8.dp.toPx()
                
                // Transform origin
                transformOrigin = TransformOrigin(0.5f, 0.5f)
            }
            .background(Color.Blue)
    )
    
    LaunchedEffect(Unit) {
        while (true) {
            delay(16)
            rotation += 1f
        }
    }
}
```

### 10. Dibujo Personalizado (DrawModifier)

```kotlin
@Composable
fun DrawModifiers() {
    // drawBehind - dibuja detrás del contenido
    Box(
        Modifier
            .size(100.dp)
            .drawBehind {
                drawCircle(
                    color = Color.Blue,
                    radius = size.minDimension / 2
                )
            }
    ) {
        Text("Texto encima")
    }
    
    // drawWithContent - control completo
    Box(
        Modifier
            .size(100.dp)
            .drawWithContent {
                // Dibujar detrás
                drawRect(Color.Gray)
                
                // Dibujar el contenido
                drawContent()
                
                // Dibujar encima
                drawLine(
                    Color.Red,
                    Offset(0f, 0f),
                    Offset(size.width, size.height),
                    strokeWidth = 4f
                )
            }
    )
    
    // drawWithCache - para optimizar cálculos
    Box(
        Modifier
            .size(100.dp)
            .drawWithCache {
                val path = Path().apply {
                    // Crear path complejo una sola vez
                    moveTo(0f, 0f)
                    lineTo(size.width, 0f)
                    lineTo(size.width / 2, size.height)
                    close()
                }
                
                onDrawBehind {
                    drawPath(path, Color.Blue)
                }
            }
    )
}
```

## Modificadores Avanzados

### 1. Semantics (Accesibilidad)

```kotlin
@Composable
fun SemanticsModifiers() {
    Box(
        Modifier
            .size(100.dp)
            .semantics {
                contentDescription = "Botón de acción"
                role = Role.Button
                onClick {
                    // Acción
                    true
                }
            }
            .clickable { /* ... */ }
    )
    
    // Merge descendants
    Row(
        Modifier.semantics(mergeDescendants = true) {
            contentDescription = "Fila de elementos"
        }
    ) {
        Text("Item 1")
        Text("Item 2")
    }
    
    // Clear semantics
    Text(
        "Decorativo",
        Modifier.clearAndSetSemantics { }
    )
}
```

### 2. Test Tags

```kotlin
@Composable
fun TestTagModifiers() {
    Button(
        onClick = { },
        modifier = Modifier.testTag("login_button")
    ) {
        Text("Login")
    }
}

// En tests
@Test
fun testLoginButton() {
    composeTestRule.onNodeWithTag("login_button").performClick()
}
```

### 3. Focus

```kotlin
@Composable
fun FocusModifiers() {
    val focusRequester = remember { FocusRequester() }
    var isFocused by remember { mutableStateOf(false) }
    
    Column {
        TextField(
            value = "",
            onValueChange = {},
            modifier = Modifier
                .focusRequester(focusRequester)
                .onFocusChanged { focusState ->
                    isFocused = focusState.isFocused
                }
                .focusable()
        )
        
        Button(
            onClick = { focusRequester.requestFocus() }
        ) {
            Text("Request Focus")
        }
        
        Text("Is Focused: $isFocused")
    }
}
```

## Patrones Comunes

### 1. Modificador Condicional

```kotlin
@Composable
fun ConditionalModifier() {
    val isSelected = remember { mutableStateOf(false) }
    
    Box(
        modifier = Modifier
            .size(100.dp)
            .then(
                if (isSelected.value) {
                    Modifier.border(2.dp, Color.Blue)
                } else {
                    Modifier
                }
            )
    )
}

// Función de extensión útil
fun Modifier.conditional(
    condition: Boolean,
    modifier: Modifier.() -> Modifier
): Modifier {
    return if (condition) {
        then(modifier(Modifier))
    } else {
        this
    }
}

// Uso
Box(
    Modifier
        .size(100.dp)
        .conditional(isSelected) {
            border(2.dp, Color.Blue)
        }
)
```

### 2. Modificador Reusable

```kotlin
// Definir modificador personalizado
fun Modifier.cardStyle() = this
    .fillMaxWidth()
    .padding(16.dp)
    .shadow(4.dp, RoundedCornerShape(8.dp))
    .background(Color.White, RoundedCornerShape(8.dp))
    .padding(16.dp)

@Composable
fun MyCard() {
    Box(Modifier.cardStyle()) {
        Text("Content")
    }
}
```

### 3. Modificador con Estado

```kotlin
@Composable
fun StatefulModifier() {
    val interactionSource = remember { MutableInteractionSource() }
    val isPressed by interactionSource.collectIsPressedAsState()
    
    Box(
        modifier = Modifier
            .size(100.dp)
            .background(if (isPressed) Color.Blue else Color.Gray)
            .clickable(
                interactionSource = interactionSource,
                indication = rememberRipple()
            ) { }
    )
}
```

## Mejores Prácticas

### 1. Orden Recomendado de Modificadores

```kotlin
@Composable
fun RecommendedOrder() {
    Box(
        Modifier
            // 1. Tamaño y layout
            .size(100.dp)
            .weight(1f)
            
            // 2. Padding y offset
            .padding(16.dp)
            .offset(x = 10.dp)
            
            // 3. Decoración visual
            .background(Color.Blue)
            .border(1.dp, Color.Red)
            
            // 4. Interacción
            .clickable { }
            
            // 5. Semántica y test tags
            .semantics { }
            .testTag("my_box")
    )
}
```

### 2. Evitar Recrear Modificadores

```kotlin
// ❌ MALO - Crea nuevo modificador en cada recomposition
@Composable
fun BadExample() {
    Box(
        Modifier
            .size(100.dp)
            .background(Color.Blue)
    )
}

// ✅ BUENO - Si el modificador es complejo, consider remember
@Composable
fun GoodExample() {
    val modifier = remember {
        Modifier
            .size(100.dp)
            .background(Color.Blue)
            // ... muchos más modificadores
    }
    
    Box(modifier)
}

// Nota: Para modificadores simples, no es necesario remember
```

### 3. Composición de Modificadores

```kotlin
// Crear modificadores base
fun Modifier.primaryButton() = this
    .height(48.dp)
    .background(Color.Blue, RoundedCornerShape(24.dp))
    .padding(horizontal = 24.dp)

fun Modifier.secondaryButton() = this
    .height(48.dp)
    .border(1.dp, Color.Blue, RoundedCornerShape(24.dp))
    .padding(horizontal = 24.dp)

@Composable
fun Buttons() {
    Row {
        Button(
            onClick = { },
            modifier = Modifier.primaryButton()
        ) {
            Text("Primary")
        }
        
        Button(
            onClick = { },
            modifier = Modifier.secondaryButton()
        ) {
            Text("Secondary")
        }
    }
}
```

## Debugging de Modificadores

```kotlin
// Extension para debug
fun Modifier.debug(tag: String = "Modifier"): Modifier {
    return this.then(
        Modifier.drawWithContent {
            drawContent()
            drawRect(
                color = Color.Red,
                style = Stroke(width = 2f)
            )
            println("$tag - Size: $size")
        }
    )
}

// Uso
Box(
    Modifier
        .size(100.dp)
        .debug("MyBox")
        .background(Color.Blue)
)
```

## Recursos

- [Compose Modifiers](https://developer.android.com/jetpack/compose/modifiers)
- [Modifier Order](https://developer.android.com/jetpack/compose/modifiers-list)
- [Graphics Modifiers](https://developer.android.com/jetpack/compose/graphics/draw/modifiers)
- [Custom Modifiers](https://developer.android.com/jetpack/compose/custom-modifiers)

---
*Los Modifiers son la base para construir UIs flexibles en Compose*
