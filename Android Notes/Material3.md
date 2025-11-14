# Material 3 (Material You) en Jetpack Compose

## Introducción a Material 3

Material 3 (también conocido como Material You) es el nuevo sistema de diseño de Google que introduce temas dinámicos, componentes renovados y mayor personalización.

## Setup

```kotlin
// En build.gradle
dependencies {
    implementation("androidx.compose.material3:material3:1.2.0")
    // Para Window Size Class
    implementation("androidx.compose.material3:material3-window-size-class:1.2.0")
    // Para iconos
    implementation("androidx.compose.material:material-icons-extended:1.6.0")
}
```

## Color Scheme

### 1. Esquemas de Color Predefinidos

```kotlin
@Composable
fun App() {
    val colorScheme = when {
        // Tema dinámico (Android 12+)
        Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (isSystemInDarkTheme()) {
                dynamicDarkColorScheme(context)
            } else {
                dynamicLightColorScheme(context)
            }
        }
        // Tema estático
        isSystemInDarkTheme() -> darkColorScheme()
        else -> lightColorScheme()
    }
    
    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}
```

### 2. Color Scheme Personalizado

```kotlin
// Definir colores personalizados
private val LightColorScheme = lightColorScheme(
    primary = Color(0xFF6750A4),
    onPrimary = Color(0xFFFFFFFF),
    primaryContainer = Color(0xFFEADDFF),
    onPrimaryContainer = Color(0xFF21005D),
    
    secondary = Color(0xFF625B71),
    onSecondary = Color(0xFFFFFFFF),
    secondaryContainer = Color(0xFFE8DEF8),
    onSecondaryContainer = Color(0xFF1D192B),
    
    tertiary = Color(0xFF7D5260),
    onTertiary = Color(0xFFFFFFFF),
    tertiaryContainer = Color(0xFFFFD8E4),
    onTertiaryContainer = Color(0xFF31111D),
    
    error = Color(0xFFB3261E),
    onError = Color(0xFFFFFFFF),
    errorContainer = Color(0xFFF9DEDC),
    onErrorContainer = Color(0xFF410E0B),
    
    background = Color(0xFFFFFBFE),
    onBackground = Color(0xFF1C1B1F),
    
    surface = Color(0xFFFFFBFE),
    onSurface = Color(0xFF1C1B1F),
    surfaceVariant = Color(0xFFE7E0EC),
    onSurfaceVariant = Color(0xFF49454F),
    
    outline = Color(0xFF79747E),
    outlineVariant = Color(0xFFCAC4D0),
    
    scrim = Color(0xFF000000),
    inverseSurface = Color(0xFF313033),
    inverseOnSurface = Color(0xFFF4EFF4),
    inversePrimary = Color(0xFFD0BCFF),
    
    surfaceTint = Color(0xFF6750A4)
)

private val DarkColorScheme = darkColorScheme(
    primary = Color(0xFFD0BCFF),
    onPrimary = Color(0xFF381E72),
    primaryContainer = Color(0xFF4F378B),
    onPrimaryContainer = Color(0xFFEADDFF),
    
    secondary = Color(0xFFCCC2DC),
    onSecondary = Color(0xFF332D41),
    secondaryContainer = Color(0xFF4A4458),
    onSecondaryContainer = Color(0xFFE8DEF8),
    
    tertiary = Color(0xFFEFB8C8),
    onTertiary = Color(0xFF492532),
    tertiaryContainer = Color(0xFF633B48),
    onTertiaryContainer = Color(0xFFFFD8E4),
    
    error = Color(0xFFF2B8B5),
    onError = Color(0xFF601410),
    errorContainer = Color(0xFF8C1D18),
    onErrorContainer = Color(0xFFF9DEDC),
    
    background = Color(0xFF1C1B1F),
    onBackground = Color(0xFFE6E1E5),
    
    surface = Color(0xFF1C1B1F),
    onSurface = Color(0xFFE6E1E5),
    surfaceVariant = Color(0xFF49454F),
    onSurfaceVariant = Color(0xFFCAC4D0),
    
    outline = Color(0xFF938F99),
    outlineVariant = Color(0xFF49454F),
    
    scrim = Color(0xFF000000),
    inverseSurface = Color(0xFFE6E1E5),
    inverseOnSurface = Color(0xFF313033),
    inversePrimary = Color(0xFF6750A4),
    
    surfaceTint = Color(0xFFD0BCFF)
)

@Composable
fun MyAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = if (darkTheme) DarkColorScheme else LightColorScheme
    
    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        shapes = Shapes,
        content = content
    )
}
```

### 3. Usar Colores del Theme

```kotlin
@Composable
fun ThemedComponents() {
    val colorScheme = MaterialTheme.colorScheme
    
    Column {
        // Usar colores del tema
        Surface(color = colorScheme.primary) {
            Text(
                "Primary Surface",
                color = colorScheme.onPrimary
            )
        }
        
        Surface(color = colorScheme.secondary) {
            Text(
                "Secondary Surface",
                color = colorScheme.onSecondary
            )
        }
        
        // Container colors
        Surface(color = colorScheme.primaryContainer) {
            Text(
                "Primary Container",
                color = colorScheme.onPrimaryContainer
            )
        }
    }
}
```

## Typography

### 1. Typography System

```kotlin
val Typography = Typography(
    // Display - Tamaños más grandes, para headlines importantes
    displayLarge = TextStyle(
        fontSize = 57.sp,
        lineHeight = 64.sp,
        fontWeight = FontWeight.Normal
    ),
    displayMedium = TextStyle(
        fontSize = 45.sp,
        lineHeight = 52.sp,
        fontWeight = FontWeight.Normal
    ),
    displaySmall = TextStyle(
        fontSize = 36.sp,
        lineHeight = 44.sp,
        fontWeight = FontWeight.Normal
    ),
    
    // Headline
    headlineLarge = TextStyle(
        fontSize = 32.sp,
        lineHeight = 40.sp,
        fontWeight = FontWeight.Normal
    ),
    headlineMedium = TextStyle(
        fontSize = 28.sp,
        lineHeight = 36.sp,
        fontWeight = FontWeight.Normal
    ),
    headlineSmall = TextStyle(
        fontSize = 24.sp,
        lineHeight = 32.sp,
        fontWeight = FontWeight.Normal
    ),
    
    // Title
    titleLarge = TextStyle(
        fontSize = 22.sp,
        lineHeight = 28.sp,
        fontWeight = FontWeight.Normal
    ),
    titleMedium = TextStyle(
        fontSize = 16.sp,
        lineHeight = 24.sp,
        fontWeight = FontWeight.Medium,
        letterSpacing = 0.15.sp
    ),
    titleSmall = TextStyle(
        fontSize = 14.sp,
        lineHeight = 20.sp,
        fontWeight = FontWeight.Medium,
        letterSpacing = 0.1.sp
    ),
    
    // Body
    bodyLarge = TextStyle(
        fontSize = 16.sp,
        lineHeight = 24.sp,
        fontWeight = FontWeight.Normal,
        letterSpacing = 0.5.sp
    ),
    bodyMedium = TextStyle(
        fontSize = 14.sp,
        lineHeight = 20.sp,
        fontWeight = FontWeight.Normal,
        letterSpacing = 0.25.sp
    ),
    bodySmall = TextStyle(
        fontSize = 12.sp,
        lineHeight = 16.sp,
        fontWeight = FontWeight.Normal,
        letterSpacing = 0.4.sp
    ),
    
    // Label
    labelLarge = TextStyle(
        fontSize = 14.sp,
        lineHeight = 20.sp,
        fontWeight = FontWeight.Medium,
        letterSpacing = 0.1.sp
    ),
    labelMedium = TextStyle(
        fontSize = 12.sp,
        lineHeight = 16.sp,
        fontWeight = FontWeight.Medium,
        letterSpacing = 0.5.sp
    ),
    labelSmall = TextStyle(
        fontSize = 11.sp,
        lineHeight = 16.sp,
        fontWeight = FontWeight.Medium,
        letterSpacing = 0.5.sp
    )
)

// Uso
@Composable
fun TypographyExample() {
    Column {
        Text("Display Large", style = MaterialTheme.typography.displayLarge)
        Text("Headline Medium", style = MaterialTheme.typography.headlineMedium)
        Text("Title Large", style = MaterialTheme.typography.titleLarge)
        Text("Body Medium", style = MaterialTheme.typography.bodyMedium)
        Text("Label Small", style = MaterialTheme.typography.labelSmall)
    }
}
```

### 2. Fuentes Personalizadas

```kotlin
val Roboto = FontFamily(
    Font(R.font.roboto_regular, FontWeight.Normal),
    Font(R.font.roboto_medium, FontWeight.Medium),
    Font(R.font.roboto_bold, FontWeight.Bold)
)

val CustomTypography = Typography(
    displayLarge = TextStyle(
        fontFamily = Roboto,
        fontWeight = FontWeight.Normal,
        fontSize = 57.sp,
        lineHeight = 64.sp
    ),
    // ... resto de estilos
)
```

## Shapes

```kotlin
val Shapes = Shapes(
    extraSmall = RoundedCornerShape(4.dp),
    small = RoundedCornerShape(8.dp),
    medium = RoundedCornerShape(12.dp),
    large = RoundedCornerShape(16.dp),
    extraLarge = RoundedCornerShape(28.dp)
)

// Uso
@Composable
fun ShapesExample() {
    val shapes = MaterialTheme.shapes
    
    Column {
        Surface(
            shape = shapes.small,
            color = MaterialTheme.colorScheme.primary
        ) {
            Text("Small shape")
        }
        
        Surface(
            shape = shapes.medium,
            color = MaterialTheme.colorScheme.secondary
        ) {
            Text("Medium shape")
        }
    }
}
```

## Componentes Material 3

### 1. Buttons

```kotlin
@Composable
fun ButtonsExample() {
    Column(
        modifier = Modifier.padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        // Filled Button (Principal)
        Button(onClick = { }) {
            Text("Filled Button")
        }
        
        // Filled Tonal Button
        FilledTonalButton(onClick = { }) {
            Text("Filled Tonal")
        }
        
        // Outlined Button
        OutlinedButton(onClick = { }) {
            Text("Outlined Button")
        }
        
        // Elevated Button
        ElevatedButton(onClick = { }) {
            Text("Elevated Button")
        }
        
        // Text Button
        TextButton(onClick = { }) {
            Text("Text Button")
        }
        
        // Icon Button
        IconButton(onClick = { }) {
            Icon(Icons.Default.Favorite, contentDescription = null)
        }
        
        // Filled Icon Button
        FilledIconButton(onClick = { }) {
            Icon(Icons.Default.Add, contentDescription = null)
        }
        
        // Outlined Icon Button
        OutlinedIconButton(onClick = { }) {
            Icon(Icons.Default.Edit, contentDescription = null)
        }
        
        // Button con icono
        Button(
            onClick = { },
            contentPadding = ButtonDefaults.ButtonWithIconContentPadding
        ) {
            Icon(
                Icons.Default.Add,
                contentDescription = null,
                modifier = Modifier.size(ButtonDefaults.IconSize)
            )
            Spacer(Modifier.size(ButtonDefaults.IconSpacing))
            Text("Add Item")
        }
    }
}
```

### 2. Cards

```kotlin
@Composable
fun CardsExample() {
    Column(
        modifier = Modifier.padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        // Card estándar
        Card(
            modifier = Modifier.fillMaxWidth()
        ) {
            Column(Modifier.padding(16.dp)) {
                Text("Card Title", style = MaterialTheme.typography.titleLarge)
                Text("Card content", style = MaterialTheme.typography.bodyMedium)
            }
        }
        
        // Elevated Card
        ElevatedCard(
            modifier = Modifier.fillMaxWidth()
        ) {
            Column(Modifier.padding(16.dp)) {
                Text("Elevated Card")
            }
        }
        
        // Outlined Card
        OutlinedCard(
            modifier = Modifier.fillMaxWidth()
        ) {
            Column(Modifier.padding(16.dp)) {
                Text("Outlined Card")
            }
        }
        
        // Card clickeable
        Card(
            onClick = { },
            modifier = Modifier.fillMaxWidth()
        ) {
            Column(Modifier.padding(16.dp)) {
                Text("Clickable Card")
            }
        }
    }
}
```

### 3. Text Fields

```kotlin
@Composable
fun TextFieldsExample() {
    var text1 by remember { mutableStateOf("") }
    var text2 by remember { mutableStateOf("") }
    
    Column(
        modifier = Modifier.padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        // TextField estándar (filled)
        TextField(
            value = text1,
            onValueChange = { text1 = it },
            label = { Text("Label") },
            placeholder = { Text("Placeholder") },
            leadingIcon = {
                Icon(Icons.Default.Email, contentDescription = null)
            },
            trailingIcon = {
                IconButton(onClick = { text1 = "" }) {
                    Icon(Icons.Default.Clear, contentDescription = null)
                }
            },
            supportingText = { Text("Supporting text") },
            isError = false
        )
        
        // Outlined TextField
        OutlinedTextField(
            value = text2,
            onValueChange = { text2 = it },
            label = { Text("Outlined") },
            singleLine = true
        )
        
        // TextField con validación
        var email by remember { mutableStateOf("") }
        val isValidEmail = android.util.Patterns.EMAIL_ADDRESS.matcher(email).matches()
        
        OutlinedTextField(
            value = email,
            onValueChange = { email = it },
            label = { Text("Email") },
            isError = email.isNotEmpty() && !isValidEmail,
            supportingText = {
                if (email.isNotEmpty() && !isValidEmail) {
                    Text("Invalid email format")
                }
            },
            keyboardOptions = KeyboardOptions(
                keyboardType = KeyboardType.Email
            )
        )
    }
}
```

### 4. Navigation Components

```kotlin
@Composable
fun NavigationExample() {
    var selectedItem by remember { mutableStateOf(0) }
    val items = listOf("Home", "Search", "Profile")
    val icons = listOf(
        Icons.Default.Home,
        Icons.Default.Search,
        Icons.Default.Person
    )
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Top App Bar") },
                navigationIcon = {
                    IconButton(onClick = { }) {
                        Icon(Icons.Default.Menu, contentDescription = null)
                    }
                },
                actions = {
                    IconButton(onClick = { }) {
                        Icon(Icons.Default.Search, contentDescription = null)
                    }
                    IconButton(onClick = { }) {
                        Icon(Icons.Default.MoreVert, contentDescription = null)
                    }
                }
            )
        },
        bottomBar = {
            NavigationBar {
                items.forEachIndexed { index, item ->
                    NavigationBarItem(
                        icon = { Icon(icons[index], contentDescription = item) },
                        label = { Text(item) },
                        selected = selectedItem == index,
                        onClick = { selectedItem = index }
                    )
                }
            }
        },
        floatingActionButton = {
            FloatingActionButton(onClick = { }) {
                Icon(Icons.Default.Add, contentDescription = "Add")
            }
        }
    ) { paddingValues ->
        Box(Modifier.padding(paddingValues)) {
            // Content
        }
    }
}
```

### 5. Navigation Rail (Para tablets/desktop)

```kotlin
@Composable
fun NavigationRailExample() {
    var selectedItem by remember { mutableStateOf(0) }
    
    Row {
        NavigationRail(
            header = {
                FloatingActionButton(
                    onClick = { },
                    modifier = Modifier.padding(vertical = 12.dp)
                ) {
                    Icon(Icons.Default.Add, contentDescription = "Add")
                }
            }
        ) {
            NavigationRailItem(
                icon = { Icon(Icons.Default.Home, contentDescription = null) },
                label = { Text("Home") },
                selected = selectedItem == 0,
                onClick = { selectedItem = 0 }
            )
            NavigationRailItem(
                icon = { Icon(Icons.Default.Search, contentDescription = null) },
                label = { Text("Search") },
                selected = selectedItem == 1,
                onClick = { selectedItem = 1 }
            )
            NavigationRailItem(
                icon = { Icon(Icons.Default.Settings, contentDescription = null) },
                label = { Text("Settings") },
                selected = selectedItem == 2,
                onClick = { selectedItem = 2 }
            )
        }
        
        // Main content
        Box(Modifier.fillMaxSize()) {
            when (selectedItem) {
                0 -> HomeContent()
                1 -> SearchContent()
                2 -> SettingsContent()
            }
        }
    }
}
```

### 6. Navigation Drawer

```kotlin
@Composable
fun NavigationDrawerExample() {
    val drawerState = rememberDrawerState(DrawerValue.Closed)
    val scope = rememberCoroutineScope()
    var selectedItem by remember { mutableStateOf(0) }
    
    ModalNavigationDrawer(
        drawerState = drawerState,
        drawerContent = {
            ModalDrawerSheet {
                Text("Drawer title", modifier = Modifier.padding(16.dp))
                Divider()
                NavigationDrawerItem(
                    icon = { Icon(Icons.Default.Home, contentDescription = null) },
                    label = { Text("Home") },
                    selected = selectedItem == 0,
                    onClick = {
                        selectedItem = 0
                        scope.launch { drawerState.close() }
                    },
                    modifier = Modifier.padding(NavigationDrawerItemDefaults.ItemPadding)
                )
                NavigationDrawerItem(
                    icon = { Icon(Icons.Default.Settings, contentDescription = null) },
                    label = { Text("Settings") },
                    selected = selectedItem == 1,
                    onClick = {
                        selectedItem = 1
                        scope.launch { drawerState.close() }
                    },
                    modifier = Modifier.padding(NavigationDrawerItemDefaults.ItemPadding)
                )
            }
        }
    ) {
        Scaffold(
            topBar = {
                TopAppBar(
                    title = { Text("Drawer Example") },
                    navigationIcon = {
                        IconButton(onClick = {
                            scope.launch { drawerState.open() }
                        }) {
                            Icon(Icons.Default.Menu, contentDescription = "Menu")
                        }
                    }
                )
            }
        ) { paddingValues ->
            Box(Modifier.padding(paddingValues)) {
                // Content
            }
        }
    }
}
```

### 7. Chips

```kotlin
@Composable
fun ChipsExample() {
    Column(
        modifier = Modifier.padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        // Assist Chip
        AssistChip(
            onClick = { },
            label = { Text("Assist Chip") },
            leadingIcon = {
                Icon(Icons.Default.Check, contentDescription = null)
            }
        )
        
        // Filter Chip
        var selected by remember { mutableStateOf(false) }
        FilterChip(
            selected = selected,
            onClick = { selected = !selected },
            label = { Text("Filter Chip") },
            leadingIcon = if (selected) {
                {
                    Icon(Icons.Default.Done, contentDescription = null)
                }
            } else null
        )
        
        // Input Chip
        InputChip(
            selected = false,
            onClick = { },
            label = { Text("Input Chip") },
            trailingIcon = {
                Icon(Icons.Default.Close, contentDescription = null)
            }
        )
        
        // Suggestion Chip
        SuggestionChip(
            onClick = { },
            label = { Text("Suggestion") }
        )
    }
}
```

### 8. Sliders y Switches

```kotlin
@Composable
fun SelectionControlsExample() {
    var sliderValue by remember { mutableStateOf(0.5f) }
    var switchState by remember { mutableStateOf(false) }
    var checkboxState by remember { mutableStateOf(false) }
    
    Column(
        modifier = Modifier.padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        // Slider
        Text("Slider: ${(sliderValue * 100).toInt()}%")
        Slider(
            value = sliderValue,
            onValueChange = { sliderValue = it }
        )
        
        // Range Slider
        var rangeValues by remember { mutableStateOf(0.3f..0.7f) }
        RangeSlider(
            value = rangeValues,
            onValueChange = { rangeValues = it },
            valueRange = 0f..1f
        )
        
        // Switch
        Row(
            verticalAlignment = Alignment.CenterVertically,
            horizontalArrangement = Arrangement.SpaceBetween,
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("Switch")
            Switch(
                checked = switchState,
                onCheckedChange = { switchState = it }
            )
        }
        
        // Checkbox
        Row(
            verticalAlignment = Alignment.CenterVertically
        ) {
            Checkbox(
                checked = checkboxState,
                onCheckedChange = { checkboxState = it }
            )
            Text("Checkbox")
        }
        
        // Radio Button
        var selectedOption by remember { mutableStateOf(0) }
        Column {
            listOf("Option 1", "Option 2", "Option 3").forEachIndexed { index, text ->
                Row(
                    verticalAlignment = Alignment.CenterVertically,
                    modifier = Modifier.clickable { selectedOption = index }
                ) {
                    RadioButton(
                        selected = selectedOption == index,
                        onClick = { selectedOption = index }
                    )
                    Text(text)
                }
            }
        }
    }
}
```

### 9. Dialogs y Sheets

```kotlin
@Composable
fun DialogsExample() {
    var showDialog by remember { mutableStateOf(false) }
    var showBottomSheet by remember { mutableStateOf(false) }
    
    Column {
        Button(onClick = { showDialog = true }) {
            Text("Show Alert Dialog")
        }
        
        Button(onClick = { showBottomSheet = true }) {
            Text("Show Bottom Sheet")
        }
    }
    
    // Alert Dialog
    if (showDialog) {
        AlertDialog(
            onDismissRequest = { showDialog = false },
            title = { Text("Dialog Title") },
            text = { Text("Dialog content goes here") },
            confirmButton = {
                TextButton(onClick = { showDialog = false }) {
                    Text("OK")
                }
            },
            dismissButton = {
                TextButton(onClick = { showDialog = false }) {
                    Text("Cancel")
                }
            }
        )
    }
    
    // Modal Bottom Sheet
    if (showBottomSheet) {
        val sheetState = rememberModalBottomSheetState()
        val scope = rememberCoroutineScope()
        
        ModalBottomSheet(
            onDismissRequest = { showBottomSheet = false },
            sheetState = sheetState
        ) {
            Column(Modifier.padding(16.dp)) {
                Text("Bottom Sheet Content", style = MaterialTheme.typography.titleLarge)
                Spacer(Modifier.height(16.dp))
                Text("More content here")
                Spacer(Modifier.height(16.dp))
                Button(
                    onClick = {
                        scope.launch {
                            sheetState.hide()
                            showBottomSheet = false
                        }
                    }
                ) {
                    Text("Close")
                }
            }
        }
    }
}
```

## Adaptive Layouts

### Window Size Class

```kotlin
@Composable
fun AdaptiveLayout() {
    val windowSizeClass = currentWindowAdaptiveInfo().windowSizeClass
    
    when (windowSizeClass.windowWidthSizeClass) {
        WindowWidthSizeClass.COMPACT -> {
            // Teléfono en portrait
            CompactLayout()
        }
        WindowWidthSizeClass.MEDIUM -> {
            // Teléfono en landscape o tablet pequeña
            MediumLayout()
        }
        WindowWidthSizeClass.EXPANDED -> {
            // Tablet grande o desktop
            ExpandedLayout()
        }
    }
}

@Composable
fun CompactLayout() {
    Scaffold(
        bottomBar = { BottomNavigationBar() }
    ) { padding ->
        // Single pane content
    }
}

@Composable
fun ExpandedLayout() {
    Row {
        NavigationRail()
        // Two or three pane content
    }
}
```

## Recursos

- [Material 3 Design](https://m3.material.io/)
- [Material 3 Compose](https://developer.android.com/jetpack/compose/designsystems/material3)
- [Color System](https://m3.material.io/styles/color/overview)
- [Typography](https://m3.material.io/styles/typography/overview)
- [Material Theme Builder](https://material-foundation.github.io/material-theme-builder/)

---
*Material 3 proporciona un sistema de diseño moderno y adaptable para Android*
