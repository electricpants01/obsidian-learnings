# Layout Composables en Jetpack Compose

## Layouts Fundamentales

### 1. Column - Disposición Vertical

```kotlin
@Composable
fun ColumnExamples() {
    // Column básico
    Column {
        Text("First")
        Text("Second")
        Text("Third")
    }
    
    // Con modificadores y alineación
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.SpaceBetween,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text("Top")
        Text("Middle")
        Text("Bottom")
    }
    
    // Vertical Arrangement opciones
    Column(verticalArrangement = Arrangement.Top) { }          // Arriba
    Column(verticalArrangement = Arrangement.Bottom) { }       // Abajo
    Column(verticalArrangement = Arrangement.Center) { }       // Centro
    Column(verticalArrangement = Arrangement.SpaceEvenly) { }  // Espaciado uniforme
    Column(verticalArrangement = Arrangement.SpaceBetween) { } // Entre elementos
    Column(verticalArrangement = Arrangement.SpaceAround) { }  // Alrededor elementos
    Column(verticalArrangement = Arrangement.spacedBy(8.dp)) { } // Espaciado fijo
    
    // Horizontal Alignment opciones
    Column(horizontalAlignment = Alignment.Start) { }          // Inicio (izquierda en LTR)
    Column(horizontalAlignment = Alignment.End) { }            // Final (derecha en LTR)
    Column(horizontalAlignment = Alignment.CenterHorizontally) { } // Centro
}
```

### 2. Row - Disposición Horizontal

```kotlin
@Composable
fun RowExamples() {
    // Row básico
    Row {
        Text("First")
        Text("Second")
        Text("Third")
    }
    
    // Con modificadores y alineación
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Icon(Icons.Default.Home, contentDescription = null)
        Text("Home")
        Icon(Icons.Default.ArrowForward, contentDescription = null)
    }
    
    // Horizontal Arrangement opciones
    Row(horizontalArrangement = Arrangement.Start) { }         // Inicio
    Row(horizontalArrangement = Arrangement.End) { }           // Final
    Row(horizontalArrangement = Arrangement.Center) { }        // Centro
    Row(horizontalArrangement = Arrangement.SpaceEvenly) { }   // Espaciado uniforme
    Row(horizontalArrangement = Arrangement.SpaceBetween) { }  // Entre elementos
    Row(horizontalArrangement = Arrangement.SpaceAround) { }   // Alrededor elementos
    Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) { } // Espaciado fijo
    
    // Vertical Alignment opciones
    Row(verticalAlignment = Alignment.Top) { }                 // Arriba
    Row(verticalAlignment = Alignment.Bottom) { }              // Abajo
    Row(verticalAlignment = Alignment.CenterVertically) { }    // Centro
}
```

### 3. Box - Superposición de Elementos

```kotlin
@Composable
fun BoxExamples() {
    // Box básico - elementos apilados
    Box {
        Image(painterResource(R.drawable.background), contentDescription = null)
        Text("Overlay Text")
    }
    
    // Con alineación
    Box(
        modifier = Modifier.size(200.dp),
        contentAlignment = Alignment.Center
    ) {
        // Fondo
        Image(
            painterResource(R.drawable.avatar),
            contentDescription = null,
            modifier = Modifier.fillMaxSize()
        )
        
        // Badge en la esquina
        Badge(
            modifier = Modifier.align(Alignment.TopEnd)
        ) {
            Text("5")
        }
        
        // Texto centrado
        Text(
            "Centered",
            modifier = Modifier.align(Alignment.Center)
        )
    }
    
    // ContentAlignment opciones
    Box(contentAlignment = Alignment.TopStart) { }
    Box(contentAlignment = Alignment.TopCenter) { }
    Box(contentAlignment = Alignment.TopEnd) { }
    Box(contentAlignment = Alignment.CenterStart) { }
    Box(contentAlignment = Alignment.Center) { }
    Box(contentAlignment = Alignment.CenterEnd) { }
    Box(contentAlignment = Alignment.BottomStart) { }
    Box(contentAlignment = Alignment.BottomCenter) { }
    Box(contentAlignment = Alignment.BottomEnd) { }
}
```

## Weight y Spacer

### 1. Weight - Distribución Proporcional

```kotlin
@Composable
fun WeightExample() {
    // En Row
    Row(Modifier.fillMaxWidth()) {
        Box(
            Modifier
                .weight(1f)  // 33.3%
                .background(Color.Red)
                .height(50.dp)
        )
        Box(
            Modifier
                .weight(1f)  // 33.3%
                .background(Color.Green)
                .height(50.dp)
        )
        Box(
            Modifier
                .weight(1f)  // 33.3%
                .background(Color.Blue)
                .height(50.dp)
        )
    }
    
    // Weights diferentes
    Row(Modifier.fillMaxWidth()) {
        Box(
            Modifier
                .weight(1f)  // 25%
                .background(Color.Red)
                .height(50.dp)
        )
        Box(
            Modifier
                .weight(3f)  // 75%
                .background(Color.Blue)
                .height(50.dp)
        )
    }
    
    // Weight con fill = false (no forzar tamaño máximo)
    Row(Modifier.fillMaxWidth()) {
        Text(
            "Short",
            Modifier.weight(1f, fill = false)
        )
        Text(
            "This is a much longer text",
            Modifier.weight(1f, fill = false)
        )
    }
    
    // En Column
    Column(Modifier.fillMaxHeight()) {
        Box(
            Modifier
                .weight(1f)
                .background(Color.Red)
                .fillMaxWidth()
        )
        Box(
            Modifier
                .weight(2f)
                .background(Color.Blue)
                .fillMaxWidth()
        )
    }
}
```

### 2. Spacer - Espacios Flexibles

```kotlin
@Composable
fun SpacerExample() {
    Column {
        Text("First")
        Spacer(modifier = Modifier.height(16.dp))  // Espacio fijo
        Text("Second")
        Spacer(modifier = Modifier.weight(1f))      // Espacio flexible
        Text("Third")
    }
    
    Row {
        Text("Left")
        Spacer(modifier = Modifier.width(16.dp))   // Espacio fijo
        Text("Right")
        Spacer(modifier = Modifier.weight(1f))     // Empuja a la derecha
        Icon(Icons.Default.ArrowForward, contentDescription = null)
    }
}
```

## LazyColumn y LazyRow

### 1. LazyColumn - Lista Vertical Eficiente

```kotlin
@Composable
fun LazyColumnExamples() {
    // Básico
    LazyColumn {
        items(100) { index ->
            Text("Item $index")
        }
    }
    
    // Con lista de datos
    val items = remember { List(100) { "Item $it" } }
    
    LazyColumn {
        items(items) { item ->
            Text(item)
        }
    }
    
    // Con key para optimización
    data class User(val id: Int, val name: String)
    val users = remember { List(100) { User(it, "User $it") } }
    
    LazyColumn {
        items(
            items = users,
            key = { user -> user.id }  // Key única para cada item
        ) { user ->
            UserCard(user)
        }
    }
    
    // Con contentPadding
    LazyColumn(
        contentPadding = PaddingValues(
            horizontal = 16.dp,
            vertical = 8.dp
        )
    ) {
        items(users) { user ->
            UserCard(user)
        }
    }
    
    // Con espaciado entre items
    LazyColumn(
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(users) { user ->
            UserCard(user)
        }
    }
    
    // Headers y diferentes tipos de items
    LazyColumn {
        item {
            HeaderSection()
        }
        
        items(users.take(5)) { user ->
            UserCard(user)
        }
        
        item {
            Divider()
        }
        
        items(users.drop(5)) { user ->
            UserCard(user)
        }
        
        item {
            FooterSection()
        }
    }
    
    // Sticky headers
    LazyColumn {
        val grouped = users.groupBy { it.name.first() }
        grouped.forEach { (initial, groupUsers) ->
            stickyHeader {
                Text(
                    text = initial.toString(),
                    modifier = Modifier
                        .fillMaxWidth()
                        .background(Color.LightGray)
                        .padding(8.dp)
                )
            }
            items(groupUsers) { user ->
                UserCard(user)
            }
        }
    }
}
```

### 2. LazyRow - Lista Horizontal Eficiente

```kotlin
@Composable
fun LazyRowExamples() {
    LazyRow(
        horizontalArrangement = Arrangement.spacedBy(8.dp),
        contentPadding = PaddingValues(horizontal = 16.dp)
    ) {
        items(20) { index ->
            Card(
                modifier = Modifier.size(150.dp)
            ) {
                Box(contentAlignment = Alignment.Center) {
                    Text("Item $index")
                }
            }
        }
    }
}
```

### 3. LazyVerticalGrid - Grilla Vertical

```kotlin
@Composable
fun LazyVerticalGridExamples() {
    // Columnas fijas
    LazyVerticalGrid(
        columns = GridCells.Fixed(2),
        contentPadding = PaddingValues(16.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(50) { index ->
            Card(
                modifier = Modifier
                    .fillMaxWidth()
                    .aspectRatio(1f)
            ) {
                Box(contentAlignment = Alignment.Center) {
                    Text("Item $index")
                }
            }
        }
    }
    
    // Columnas adaptativas (ancho mínimo)
    LazyVerticalGrid(
        columns = GridCells.Adaptive(minSize = 100.dp)
    ) {
        items(50) { index ->
            GridItem(index)
        }
    }
    
    // Items de diferentes tamaños con span
    LazyVerticalGrid(
        columns = GridCells.Fixed(3)
    ) {
        item(span = { GridItemSpan(3) }) {
            HeaderCard()
        }
        
        items(20) { index ->
            if (index % 7 == 0) {
                // Item grande
                item(span = { GridItemSpan(2) }) {
                    LargeCard(index)
                }
            } else {
                SmallCard(index)
            }
        }
    }
}
```

### 4. LazyHorizontalGrid - Grilla Horizontal

```kotlin
@Composable
fun LazyHorizontalGridExample() {
    LazyHorizontalGrid(
        rows = GridCells.Fixed(2),
        contentPadding = PaddingValues(16.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp),
        modifier = Modifier.height(200.dp)
    ) {
        items(20) { index ->
            Card(
                modifier = Modifier.size(100.dp)
            ) {
                Box(contentAlignment = Alignment.Center) {
                    Text("$index")
                }
            }
        }
    }
}
```

## LazyList State

### 1. Control de Scroll

```kotlin
@Composable
fun LazyListStateExample() {
    val listState = rememberLazyListState()
    val coroutineScope = rememberCoroutineScope()
    
    Column {
        // Botones de control
        Row {
            Button(onClick = {
                coroutineScope.launch {
                    listState.animateScrollToItem(0)
                }
            }) {
                Text("Scroll to top")
            }
            
            Button(onClick = {
                coroutineScope.launch {
                    listState.scrollToItem(50)
                }
            }) {
                Text("Jump to 50")
            }
        }
        
        // Estado del scroll
        val firstVisibleIndex by remember {
            derivedStateOf { listState.firstVisibleItemIndex }
        }
        
        Text("First visible: $firstVisibleIndex")
        
        // Lista
        LazyColumn(state = listState) {
            items(100) { index ->
                Text(
                    "Item $index",
                    Modifier.padding(16.dp)
                )
            }
        }
    }
}
```

### 2. Infinite Scroll / Paginación

```kotlin
@Composable
fun InfiniteScrollExample() {
    val listState = rememberLazyListState()
    val items = remember { mutableStateListOf(*(0..20).toList().toTypedArray()) }
    
    // Detectar cuando llegamos al final
    LaunchedEffect(listState) {
        snapshotFlow { listState.layoutInfo.visibleItemsInfo.lastOrNull() }
            .collect { lastVisible ->
                if (lastVisible != null && lastVisible.index >= items.size - 3) {
                    // Cargar más items
                    val currentSize = items.size
                    items.addAll((currentSize until currentSize + 20).toList())
                }
            }
    }
    
    LazyColumn(state = listState) {
        items(items) { item ->
            Text("Item $item", Modifier.padding(16.dp))
        }
    }
}
```

### 3. Mostrar/Ocultar FAB según Scroll

```kotlin
@Composable
fun ScrollAwareFAB() {
    val listState = rememberLazyListState()
    
    // FAB visible solo cuando no está en el top
    val showFab by remember {
        derivedStateOf {
            listState.firstVisibleItemIndex > 0
        }
    }
    
    Scaffold(
        floatingActionButton = {
            AnimatedVisibility(visible = showFab) {
                FloatingActionButton(
                    onClick = { /* Scroll to top */ }
                ) {
                    Icon(Icons.Default.KeyboardArrowUp, contentDescription = null)
                }
            }
        }
    ) { padding ->
        LazyColumn(
            state = listState,
            modifier = Modifier.padding(padding)
        ) {
            items(100) { index ->
                Text("Item $index", Modifier.padding(16.dp))
            }
        }
    }
}
```

## FlowRow y FlowColumn (Compose 1.4+)

### 1. FlowRow - Wrapping Horizontal

```kotlin
@Composable
fun FlowRowExample() {
    // Flow básico - los items hacen wrap a la siguiente línea
    FlowRow(
        modifier = Modifier.fillMaxWidth(),
        horizontalArrangement = Arrangement.spacedBy(8.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp),
        maxItemsInEachRow = 3
    ) {
        val tags = listOf("Kotlin", "Compose", "Android", "Jetpack", "Material")
        tags.forEach { tag ->
            AssistChip(
                onClick = { },
                label = { Text(tag) }
            )
        }
    }
    
    // Con max items por fila
    FlowRow(
        maxItemsInEachRow = 4,
        horizontalArrangement = Arrangement.spacedBy(4.dp)
    ) {
        repeat(20) { index ->
            Box(
                modifier = Modifier
                    .size(60.dp)
                    .background(Color.Blue)
            )
        }
    }
}
```

### 2. FlowColumn - Wrapping Vertical

```kotlin
@Composable
fun FlowColumnExample() {
    FlowColumn(
        modifier = Modifier
            .fillMaxHeight()
            .width(200.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp),
        maxItemsInEachColumn = 5
    ) {
        repeat(20) { index ->
            Text("Item $index")
        }
    }
}
```

## ConstraintLayout en Compose

```kotlin
@Composable
fun ConstraintLayoutExample() {
    ConstraintLayout(
        modifier = Modifier.fillMaxSize()
    ) {
        // Crear referencias
        val (button, text, image) = createRefs()
        
        Button(
            onClick = { },
            modifier = Modifier.constrainAs(button) {
                top.linkTo(parent.top, margin = 16.dp)
                start.linkTo(parent.start, margin = 16.dp)
            }
        ) {
            Text("Click Me")
        }
        
        Text(
            "Hello World",
            modifier = Modifier.constrainAs(text) {
                top.linkTo(button.bottom, margin = 16.dp)
                centerHorizontallyTo(parent)
            }
        )
        
        Image(
            painterResource(R.drawable.image),
            contentDescription = null,
            modifier = Modifier.constrainAs(image) {
                bottom.linkTo(parent.bottom, margin = 16.dp)
                end.linkTo(parent.end, margin = 16.dp)
                width = Dimension.value(100.dp)
                height = Dimension.value(100.dp)
            }
        )
    }
    
    // Chains
    ConstraintLayout(Modifier.fillMaxSize()) {
        val (first, second, third) = createRefs()
        
        createHorizontalChain(first, second, third, chainStyle = ChainStyle.Spread)
        
        Box(Modifier.constrainAs(first) {
            centerVerticallyTo(parent)
        }.size(50.dp).background(Color.Red))
        
        Box(Modifier.constrainAs(second) {
            centerVerticallyTo(parent)
        }.size(50.dp).background(Color.Green))
        
        Box(Modifier.constrainAs(third) {
            centerVerticallyTo(parent)
        }.size(50.dp).background(Color.Blue))
    }
    
    // Guidelines
    ConstraintLayout(Modifier.fillMaxSize()) {
        val (text) = createRefs()
        val guideline = createGuidelineFromTop(0.3f)
        
        Text(
            "Centered on guideline",
            modifier = Modifier.constrainAs(text) {
                top.linkTo(guideline)
                centerHorizontallyTo(parent)
            }
        )
    }
}
```

## Scaffold y Layouts de Screen

```kotlin
@Composable
fun ScaffoldExample() {
    val snackbarHostState = remember { SnackbarHostState() }
    val scope = rememberCoroutineScope()
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("My App") },
                navigationIcon = {
                    IconButton(onClick = { }) {
                        Icon(Icons.Default.Menu, contentDescription = null)
                    }
                }
            )
        },
        bottomBar = {
            NavigationBar {
                NavigationBarItem(
                    selected = true,
                    onClick = { },
                    icon = { Icon(Icons.Default.Home, contentDescription = null) },
                    label = { Text("Home") }
                )
                NavigationBarItem(
                    selected = false,
                    onClick = { },
                    icon = { Icon(Icons.Default.Search, contentDescription = null) },
                    label = { Text("Search") }
                )
            }
        },
        floatingActionButton = {
            FloatingActionButton(
                onClick = {
                    scope.launch {
                        snackbarHostState.showSnackbar("FAB clicked!")
                    }
                }
            ) {
                Icon(Icons.Default.Add, contentDescription = null)
            }
        },
        snackbarHost = {
            SnackbarHost(hostState = snackbarHostState)
        }
    ) { paddingValues ->
        // Contenido principal
        LazyColumn(
            modifier = Modifier.padding(paddingValues)
        ) {
            items(50) { index ->
                Text("Item $index", Modifier.padding(16.dp))
            }
        }
    }
}
```

## Mejores Prácticas

### 1. Elegir el Layout Correcto

```kotlin
// ✅ BUENO - Usar LazyColumn para listas largas
@Composable
fun LongList() {
    LazyColumn {
        items(1000) { index ->
            Text("Item $index")
        }
    }
}

// ❌ MALO - Column regular para muchos items
@Composable
fun BadLongList() {
    Column {
        repeat(1000) { index ->  // Todos se crean de una vez
            Text("Item $index")
        }
    }
}
```

### 2. Keys en LazyLists

```kotlin
// ✅ BUENO - Usar keys únicas
LazyColumn {
    items(
        items = users,
        key = { user -> user.id }
    ) { user ->
        UserCard(user)
    }
}

// ❌ MALO - Sin keys
LazyColumn {
    items(users) { user ->
        UserCard(user)
    }
}
```

### 3. ContentPadding vs Modifier.padding

```kotlin
// ✅ BUENO - contentPadding para LazyLists
LazyColumn(
    contentPadding = PaddingValues(16.dp)
) {
    items(items) { item ->
        ItemCard(item)
    }
}

// ❌ MALO - padding en el modifier (items se cortan)
LazyColumn(
    modifier = Modifier.padding(16.dp)
) {
    items(items) { item ->
        ItemCard(item)
    }
}
```

## Recursos

- [Compose Layouts](https://developer.android.com/jetpack/compose/layouts)
- [Lazy Layouts](https://developer.android.com/jetpack/compose/lists)
- [Custom Layouts](https://developer.android.com/jetpack/compose/layouts/custom)
- [ConstraintLayout](https://developer.android.com/jetpack/compose/layouts/constraintlayout)

---
*Los layouts son la base para estructurar UIs efectivas en Compose*
