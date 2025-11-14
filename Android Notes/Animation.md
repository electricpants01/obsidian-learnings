# Animaciones en Jetpack Compose

## Tipos de Animaciones en Compose

Compose ofrece múltiples APIs de animación para diferentes casos de uso, desde simples cambios de valor hasta transiciones complejas.

## animate*AsState - Animaciones Simples

### 1. Animaciones de Valores

```kotlin
@Composable
fun AnimateAsStateExamples() {
    var isExpanded by remember { mutableStateOf(false) }
    
    // Animar Float
    val alpha by animateFloatAsState(
        targetValue = if (isExpanded) 1f else 0.5f,
        label = "alpha animation"
    )
    
    // Animar Dp
    val size by animateDpAsState(
        targetValue = if (isExpanded) 200.dp else 100.dp,
        label = "size animation"
    )
    
    // Animar Color
    val color by animateColorAsState(
        targetValue = if (isExpanded) Color.Blue else Color.Red,
        label = "color animation"
    )
    
    // Animar Int
    val count by animateIntAsState(
        targetValue = if (isExpanded) 100 else 0,
        label = "count animation"
    )
    
    Box(
        modifier = Modifier
            .size(size)
            .alpha(alpha)
            .background(color)
            .clickable { isExpanded = !isExpanded },
        contentAlignment = Alignment.Center
    ) {
        Text("Count: $count")
    }
}
```

### 2. AnimationSpec - Configurar Animaciones

```kotlin
@Composable
fun AnimationSpecExamples() {
    var enabled by remember { mutableStateOf(false) }
    
    // Spring - Físicamente realista con rebote
    val springSize by animateDpAsState(
        targetValue = if (enabled) 200.dp else 100.dp,
        animationSpec = spring(
            dampingRatio = Spring.DampingRatioMediumBouncy,
            stiffness = Spring.StiffnessLow
        ),
        label = "spring"
    )
    
    // Tween - Interpolación con duración fija
    val tweenSize by animateDpAsState(
        targetValue = if (enabled) 200.dp else 100.dp,
        animationSpec = tween(
            durationMillis = 1000,
            delayMillis = 100,
            easing = FastOutSlowInEasing
        ),
        label = "tween"
    )
    
    // Keyframes - Animación con pasos específicos
    val keyframesSize by animateDpAsState(
        targetValue = if (enabled) 200.dp else 100.dp,
        animationSpec = keyframes {
            durationMillis = 1000
            100.dp at 0 with LinearEasing
            150.dp at 500 with FastOutSlowInEasing
            200.dp at 1000
        },
        label = "keyframes"
    )
    
    // Repeatable - Repetir animación
    val repeatableAlpha by animateFloatAsState(
        targetValue = if (enabled) 1f else 0.3f,
        animationSpec = repeatable(
            iterations = 3,
            animation = tween(500),
            repeatMode = RepeatMode.Reverse
        ),
        label = "repeatable"
    )
    
    // InfiniteRepeatable - Animación infinita
    val infiniteTransition = rememberInfiniteTransition(label = "infinite")
    val infiniteAlpha by infiniteTransition.animateFloat(
        initialValue = 0.3f,
        targetValue = 1f,
        animationSpec = infiniteRepeatable(
            animation = tween(1000),
            repeatMode = RepeatMode.Reverse
        ),
        label = "infinite alpha"
    )
    
    Column {
        Box(Modifier.size(springSize).background(Color.Red))
        Box(Modifier.size(tweenSize).background(Color.Green))
        Box(Modifier.size(keyframesSize).background(Color.Blue))
        Box(
            Modifier
                .size(100.dp)
                .alpha(infiniteAlpha)
                .background(Color.Yellow)
        )
        
        Button(onClick = { enabled = !enabled }) {
            Text("Toggle")
        }
    }
}
```

## Animatable - Animaciones Imperativas

```kotlin
@Composable
fun AnimatableExample() {
    val color = remember { Animatable(Color.Red) }
    val coroutineScope = rememberCoroutineScope()
    
    LaunchedEffect(Unit) {
        // Secuencia de animaciones
        color.animateTo(Color.Blue, animationSpec = tween(1000))
        color.animateTo(Color.Green, animationSpec = tween(1000))
        color.animateTo(Color.Red, animationSpec = tween(1000))
    }
    
    Box(
        Modifier
            .size(200.dp)
            .background(color.value)
            .clickable {
                coroutineScope.launch {
                    // Animar al hacer click
                    color.animateTo(
                        targetValue = Color(
                            red = Random.nextFloat(),
                            green = Random.nextFloat(),
                            blue = Random.nextFloat()
                        ),
                        animationSpec = spring(
                            dampingRatio = Spring.DampingRatioMediumBouncy
                        )
                    )
                }
            }
    )
}

@Composable
fun AnimatableDecayExample() {
    val offsetX = remember { Animatable(0f) }
    val coroutineScope = rememberCoroutineScope()
    
    Box(
        Modifier
            .fillMaxWidth()
            .height(200.dp)
            .pointerInput(Unit) {
                detectDragGestures(
                    onDrag = { change, dragAmount ->
                        change.consume()
                        coroutineScope.launch {
                            offsetX.snapTo(offsetX.value + dragAmount.x)
                        }
                    },
                    onDragEnd = {
                        coroutineScope.launch {
                            // Decay animation - desaceleración natural
                            offsetX.animateDecay(
                                initialVelocity = 2000f,
                                animationSpec = exponentialDecay()
                            )
                        }
                    }
                )
            }
    ) {
        Box(
            Modifier
                .size(100.dp)
                .offset { IntOffset(offsetX.value.roundToInt(), 0) }
                .background(Color.Blue)
        )
    }
}
```

## updateTransition - Múltiples Valores Animados

```kotlin
enum class BoxState {
    Small, Large
}

@Composable
fun UpdateTransitionExample() {
    var boxState by remember { mutableStateOf(BoxState.Small) }
    val transition = updateTransition(
        targetState = boxState,
        label = "box transition"
    )
    
    // Animar múltiples propiedades coordinadas
    val size by transition.animateDp(
        label = "size",
        transitionSpec = {
            when {
                BoxState.Small isTransitioningTo BoxState.Large ->
                    spring(stiffness = Spring.StiffnessLow)
                else ->
                    tween(durationMillis = 300)
            }
        }
    ) { state ->
        when (state) {
            BoxState.Small -> 100.dp
            BoxState.Large -> 200.dp
        }
    }
    
    val color by transition.animateColor(
        label = "color"
    ) { state ->
        when (state) {
            BoxState.Small -> Color.Red
            BoxState.Large -> Color.Blue
        }
    }
    
    val cornerRadius by transition.animateDp(
        label = "corner"
    ) { state ->
        when (state) {
            BoxState.Small -> 0.dp
            BoxState.Large -> 50.dp
        }
    }
    
    Box(
        Modifier
            .size(size)
            .background(color, RoundedCornerShape(cornerRadius))
            .clickable {
                boxState = when (boxState) {
                    BoxState.Small -> BoxState.Large
                    BoxState.Large -> BoxState.Small
                }
            }
    )
}
```

## AnimatedVisibility - Aparecer/Desaparecer

```kotlin
@Composable
fun AnimatedVisibilityExamples() {
    var visible by remember { mutableStateOf(false) }
    
    Column {
        Button(onClick = { visible = !visible }) {
            Text("Toggle")
        }
        
        // Básico
        AnimatedVisibility(visible = visible) {
            Text("Hello World!")
        }
        
        // Con animación personalizada
        AnimatedVisibility(
            visible = visible,
            enter = fadeIn() + slideInVertically(),
            exit = fadeOut() + slideOutVertically()
        ) {
            Text("Custom Animation")
        }
        
        // Diferentes animaciones
        AnimatedVisibility(
            visible = visible,
            enter = slideInHorizontally(
                initialOffsetX = { fullWidth -> fullWidth },
                animationSpec = tween(300)
            ) + fadeIn(),
            exit = slideOutHorizontally(
                targetOffsetX = { fullWidth -> -fullWidth },
                animationSpec = tween(300)
            ) + fadeOut()
        ) {
            Text("Slide from right")
        }
        
        // Scale animation
        AnimatedVisibility(
            visible = visible,
            enter = scaleIn(
                initialScale = 0f,
                animationSpec = spring(
                    dampingRatio = Spring.DampingRatioMediumBouncy
                )
            ) + fadeIn(),
            exit = scaleOut() + fadeOut()
        ) {
            Box(
                Modifier
                    .size(100.dp)
                    .background(Color.Blue)
            )
        }
        
        // Expand/Collapse
        AnimatedVisibility(
            visible = visible,
            enter = expandVertically(),
            exit = shrinkVertically()
        ) {
            Column {
                repeat(5) { index ->
                    Text("Item $index")
                }
            }
        }
    }
}
```

### AnimatedVisibility con Children

```kotlin
@Composable
fun AnimatedVisibilityChildren() {
    var visible by remember { mutableStateOf(false) }
    
    AnimatedVisibility(
        visible = visible,
        enter = fadeIn() + expandVertically(),
        exit = fadeOut() + shrinkVertically()
    ) {
        Column {
            // Animar children individualmente
            Text(
                "Title",
                modifier = Modifier.animateEnterExit(
                    enter = slideInVertically(),
                    exit = slideOutVertically()
                )
            )
            
            Text(
                "Subtitle",
                modifier = Modifier.animateEnterExit(
                    enter = slideInVertically(
                        initialOffsetY = { it },
                        animationSpec = tween(delayMillis = 100)
                    ),
                    exit = slideOutVertically()
                )
            )
        }
    }
}
```

## AnimatedContent - Transiciones de Contenido

```kotlin
@Composable
fun AnimatedContentExample() {
    var count by remember { mutableStateOf(0) }
    
    Column {
        Button(onClick = { count++ }) {
            Text("Increment")
        }
        
        // Básico
        AnimatedContent(
            targetState = count,
            label = "count animation"
        ) { targetCount ->
            Text("Count: $targetCount")
        }
        
        // Con transición personalizada
        AnimatedContent(
            targetState = count,
            transitionSpec = {
                // Slide up and fade in
                slideInVertically { height -> height } + fadeIn() togetherWith
                    slideOutVertically { height -> -height } + fadeOut()
            },
            label = "count animation"
        ) { targetCount ->
            Text(
                text = "$targetCount",
                style = MaterialTheme.typography.displayLarge
            )
        }
        
        // Size transform
        AnimatedContent(
            targetState = count,
            transitionSpec = {
                fadeIn() + scaleIn(initialScale = 0.8f) togetherWith
                    fadeOut() + scaleOut(targetScale = 1.2f) using
                    SizeTransform { initialSize, targetSize ->
                        keyframes {
                            durationMillis = 300
                            initialSize at 0
                            targetSize at 300
                        }
                    }
            },
            label = "animated content"
        ) { targetCount ->
            Box(
                Modifier
                    .size(100.dp)
                    .background(Color.Blue),
                contentAlignment = Alignment.Center
            ) {
                Text("$targetCount", color = Color.White)
            }
        }
    }
}
```

## Crossfade

```kotlin
@Composable
fun CrossfadeExample() {
    var currentPage by remember { mutableStateOf("A") }
    
    Column {
        Row {
            Button(onClick = { currentPage = "A" }) { Text("A") }
            Button(onClick = { currentPage = "B" }) { Text("B") }
            Button(onClick = { currentPage = "C" }) { Text("C") }
        }
        
        Crossfade(
            targetState = currentPage,
            animationSpec = tween(1000),
            label = "crossfade"
        ) { page ->
            when (page) {
                "A" -> PageA()
                "B" -> PageB()
                "C" -> PageC()
            }
        }
    }
}
```

## Animaciones Gestuales

### 1. Swipe to Dismiss

```kotlin
@Composable
fun SwipeToDismissExample() {
    val dismissState = rememberSwipeToDismissBoxState(
        confirmValueChange = { dismissValue ->
            when (dismissValue) {
                SwipeToDismissBoxValue.StartToEnd -> {
                    // Acción al swipe derecha
                    true
                }
                SwipeToDismissBoxValue.EndToStart -> {
                    // Acción al swipe izquierda
                    true
                }
                else -> false
            }
        }
    )
    
    SwipeToDismissBox(
        state = dismissState,
        backgroundContent = {
            val color by animateColorAsState(
                when (dismissState.targetValue) {
                    SwipeToDismissBoxValue.StartToEnd -> Color.Green
                    SwipeToDismissBoxValue.EndToStart -> Color.Red
                    else -> Color.Gray
                },
                label = "background color"
            )
            
            Box(
                Modifier
                    .fillMaxSize()
                    .background(color)
                    .padding(16.dp)
            ) {
                Icon(
                    Icons.Default.Delete,
                    contentDescription = null,
                    modifier = Modifier.align(Alignment.CenterEnd)
                )
            }
        }
    ) {
        Card(
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("Swipe me", Modifier.padding(16.dp))
        }
    }
}
```

### 2. Draggable con Animación

```kotlin
@Composable
fun DraggableWithAnimation() {
    var offsetX by remember { mutableStateOf(0f) }
    val maxOffset = 300f
    
    Box(
        Modifier
            .fillMaxSize()
            .draggable(
                orientation = Orientation.Horizontal,
                state = rememberDraggableState { delta ->
                    offsetX = (offsetX + delta).coerceIn(-maxOffset, maxOffset)
                },
                onDragStopped = { velocity ->
                    // Snap back con animación
                    animate(
                        initialValue = offsetX,
                        targetValue = 0f,
                        initialVelocity = velocity
                    ) { value, _ ->
                        offsetX = value
                    }
                }
            )
    ) {
        Box(
            Modifier
                .size(100.dp)
                .offset { IntOffset(offsetX.roundToInt(), 0) }
                .background(Color.Blue)
        )
    }
}
```

## Animaciones de Lista

### 1. Animar Items en LazyColumn

```kotlin
@Composable
fun AnimatedLazyColumn() {
    var items by remember {
        mutableStateOf(List(20) { "Item $it" })
    }
    
    Column {
        Button(onClick = {
            items = items + "New Item ${items.size}"
        }) {
            Text("Add Item")
        }
        
        LazyColumn {
            items(
                items = items,
                key = { it }
            ) { item ->
                AnimatedListItem(
                    text = item,
                    onDelete = { items = items - item }
                )
            }
        }
    }
}

@Composable
fun AnimatedListItem(
    text: String,
    onDelete: () -> Unit
) {
    var visible by remember { mutableStateOf(false) }
    
    LaunchedEffect(Unit) {
        visible = true
    }
    
    AnimatedVisibility(
        visible = visible,
        enter = slideInVertically() + expandVertically() + fadeIn(),
        exit = slideOutVertically() + shrinkVertically() + fadeOut()
    ) {
        Card(
            modifier = Modifier
                .fillMaxWidth()
                .padding(8.dp)
                .animateItemPlacement()
        ) {
            Row(
                modifier = Modifier.padding(16.dp),
                horizontalArrangement = Arrangement.SpaceBetween
            ) {
                Text(text)
                IconButton(onClick = {
                    visible = false
                    // Delay para la animación
                    kotlinx.coroutines.delay(300)
                    onDelete()
                }) {
                    Icon(Icons.Default.Delete, contentDescription = "Delete")
                }
            }
        }
    }
}
```

## Animaciones Complejas

### 1. Morphing Shapes

```kotlin
@Composable
fun MorphingShapes() {
    var isCir by remember { mutableStateOf(true) }
    val shape by animateIntAsState(
        targetValue = if (isCircle) 50 else 0,
        label = "shape animation"
    )
    
    Box(
        Modifier
            .size(200.dp)
            .background(
                Color.Blue,
                RoundedCornerShape(shape)
            )
            .clickable { isCircle = !isCircle }
    )
}
```

### 2. Loading Animation

```kotlin
@Composable
fun LoadingAnimation() {
    val infiniteTransition = rememberInfiniteTransition(label = "loading")
    
    val rotation by infiniteTransition.animateFloat(
        initialValue = 0f,
        targetValue = 360f,
        animationSpec = infiniteRepeatable(
            animation = tween(1000, easing = LinearEasing),
            repeatMode = RepeatMode.Restart
        ),
        label = "rotation"
    )
    
    val scale by infiniteTransition.animateFloat(
        initialValue = 0.8f,
        targetValue = 1.2f,
        animationSpec = infiniteRepeatable(
            animation = tween(500),
            repeatMode = RepeatMode.Reverse
        ),
        label = "scale"
    )
    
    Box(
        Modifier
            .size(100.dp)
            .graphicsLayer {
                rotationZ = rotation
                scaleX = scale
                scaleY = scale
            }
            .background(Color.Blue, CircleShape)
    )
}
```

### 3. Shimmer Effect

```kotlin
@Composable
fun ShimmerEffect() {
    val infiniteTransition = rememberInfiniteTransition(label = "shimmer")
    val shimmerTranslate by infiniteTransition.animateFloat(
        initialValue = 0f,
        targetValue = 1000f,
        animationSpec = infiniteRepeatable(
            animation = tween(1000, easing = LinearEasing),
            repeatMode = RepeatMode.Restart
        ),
        label = "shimmer translate"
    )
    
    Box(
        Modifier
            .fillMaxWidth()
            .height(200.dp)
            .background(
                brush = Brush.linearGradient(
                    colors = listOf(
                        Color.LightGray.copy(alpha = 0.6f),
                        Color.LightGray.copy(alpha = 0.2f),
                        Color.LightGray.copy(alpha = 0.6f)
                    ),
                    start = Offset(shimmerTranslate - 1000f, 0f),
                    end = Offset(shimmerTranslate, 0f)
                )
            )
    )
}
```

## Animaciones de Navegación

```kotlin
// Ver Navigation-Compose.md para ejemplos de animaciones de navegación
```

## Performance de Animaciones

### 1. Usar GraphicsLayer

```kotlin
// ✅ BUENO - graphicsLayer es eficiente
@Composable
fun EfficientAnimation() {
    var scale by remember { mutableStateOf(1f) }
    
    Box(
        Modifier
            .size(100.dp)
            .graphicsLayer {
                scaleX = scale
                scaleY = scale
            }
            .background(Color.Blue)
    )
}

// ❌ MALO - Modifier.scale causa recomposition
@Composable
fun InefficientAnimation() {
    var scale by remember { mutableStateOf(1f) }
    
    Box(
        Modifier
            .size(100.dp * scale)  // Recompone layout
            .background(Color.Blue)
    )
}
```

### 2. Remember Animation Specs

```kotlin
// ✅ BUENO
@Composable
fun RememberedSpec() {
    val spec = remember {
        spring<Dp>(
            dampingRatio = Spring.DampingRatioMediumBouncy,
            stiffness = Spring.StiffnessLow
        )
    }
    
    val size by animateDpAsState(100.dp, spec, label = "size")
}
```

## Recursos

- [Compose Animation](https://developer.android.com/jetpack/compose/animation)
- [Animation Specs](https://developer.android.com/jetpack/compose/animation/customize)
- [Animate as State](https://developer.android.com/jetpack/compose/animation/value-based)
- [AnimatedVisibility](https://developer.android.com/jetpack/compose/animation/composables-modifiers)

---
*Las animaciones dan vida a la UI y mejoran la experiencia del usuario*
