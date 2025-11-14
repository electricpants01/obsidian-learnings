# Theming en Jetpack Compose

## Sistema de Theming en Compose

El theming en Compose se basa en **MaterialTheme** que provee colores, tipografía y formas a todos los componentes de la app.

## Estructura del Theme

```kotlin
@Composable
fun MyAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }
    
    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        shapes = Shapes,
        content = content
    )
}
```

## Color System Avanzado

### 1. Definir Paletas de Color Extendidas

```kotlin
// Colores adicionales personalizados
data class ExtendedColors(
    val success: Color,
    val onSuccess: Color,
    val successContainer: Color,
    val onSuccessContainer: Color,
    val warning: Color,
    val onWarning: Color,
    val info: Color,
    val onInfo: Color
)

val LightExtendedColors = ExtendedColors(
    success = Color(0xFF4CAF50),
    onSuccess = Color(0xFFFFFFFF),
    successContainer = Color(0xFFC8E6C9),
    onSuccessContainer = Color(0xFF1B5E20),
    warning = Color(0xFFFF9800),
    onWarning = Color(0xFF000000),
    info = Color(0xFF2196F3),
    onInfo = Color(0xFFFFFFFF)
)

val DarkExtendedColors = ExtendedColors(
    success = Color(0xFF81C784),
    onSuccess = Color(0xFF000000),
    successContainer = Color(0xFF2E7D32),
    onSuccessContainer = Color(0xFFC8E6C9),
    warning = Color(0xFFFFB74D),
    onWarning = Color(0xFF000000),
    info = Color(0xFF64B5F6),
    onInfo = Color(0xFF000000)
)

// CompositionLocal para colores extendidos
val LocalExtendedColors = staticCompositionLocalOf {
    LightExtendedColors
}

@Composable
fun MyAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val extendedColors = if (darkTheme) DarkExtendedColors else LightExtendedColors
    
    CompositionLocalProvider(LocalExtendedColors provides extendedColors) {
        MaterialTheme(
            colorScheme = if (darkTheme) DarkColorScheme else LightColorScheme,
            content = content
        )
    }
}

// Extension para acceder a colores extendidos
val MaterialTheme.extendedColors: ExtendedColors
    @Composable
    @ReadOnlyComposable
    get() = LocalExtendedColors.current

// Uso
@Composable
fun SuccessButton() {
    Button(
        onClick = { },
        colors = ButtonDefaults.buttonColors(
            containerColor = MaterialTheme.extendedColors.success,
            contentColor = MaterialTheme.extendedColors.onSuccess
        )
    ) {
        Text("Success")
    }
}
```

### 2. Color Roles y Semantic Colors

```kotlin
// Colores semánticos basados en contexto
object AppColors {
    val ColorScheme.cardBackground: Color
        @Composable
        get() = surfaceVariant
    
    val ColorScheme.cardBorder: Color
        @Composable
        get() = outline
    
    val ColorScheme.textPrimary: Color
        @Composable
        get() = onSurface
    
    val ColorScheme.textSecondary: Color
        @Composable
        get() = onSurfaceVariant
    
    val ColorScheme.iconPrimary: Color
        @Composable
        get() = onSurface
    
    val ColorScheme.divider: Color
        @Composable
        get() = outlineVariant
}

// Uso
@Composable
fun SemanticColorsExample() {
    val colors = MaterialTheme.colorScheme
    
    Surface(
        color = AppColors.run { colors.cardBackground },
        border = BorderStroke(1.dp, AppColors.run { colors.cardBorder })
    ) {
        Column {
            Text(
                "Primary Text",
                color = AppColors.run { colors.textPrimary }
            )
            Text(
                "Secondary Text",
                color = AppColors.run { colors.textSecondary }
            )
        }
    }
}
```

## Typography Avanzada

### 1. Typography con Múltiples Font Families

```kotlin
// Definir múltiples font families
val HeadingFontFamily = FontFamily(
    Font(R.font.playfair_display_regular, FontWeight.Normal),
    Font(R.font.playfair_display_bold, FontWeight.Bold)
)

val BodyFontFamily = FontFamily(
    Font(R.font.roboto_regular, FontWeight.Normal),
    Font(R.font.roboto_medium, FontWeight.Medium),
    Font(R.font.roboto_bold, FontWeight.Bold)
)

val MonoFontFamily = FontFamily(
    Font(R.font.jetbrains_mono_regular, FontWeight.Normal)
)

val AppTypography = Typography(
    // Headings usan Playfair Display
    displayLarge = TextStyle(
        fontFamily = HeadingFontFamily,
        fontWeight = FontWeight.Bold,
        fontSize = 57.sp,
        lineHeight = 64.sp
    ),
    headlineMedium = TextStyle(
        fontFamily = HeadingFontFamily,
        fontWeight = FontWeight.Bold,
        fontSize = 28.sp,
        lineHeight = 36.sp
    ),
    
    // Body usa Roboto
    bodyLarge = TextStyle(
        fontFamily = BodyFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp
    ),
    bodyMedium = TextStyle(
        fontFamily = BodyFontFamily,
        fontWeight = FontWeight.Normal,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.25.sp
    )
)

// Typography extensions para casos especiales
object AppTextStyles {
    val Typography.code: TextStyle
        @Composable
        get() = TextStyle(
            fontFamily = MonoFontFamily,
            fontSize = 14.sp,
            lineHeight = 20.sp,
            background = MaterialTheme.colorScheme.surfaceVariant,
            color = MaterialTheme.colorScheme.onSurfaceVariant
        )
    
    val Typography.quote: TextStyle
        @Composable
        get() = bodyLarge.copy(
            fontStyle = FontStyle.Italic,
            color = MaterialTheme.colorScheme.onSurfaceVariant
        )
}
```

### 2. Responsive Typography

```kotlin
@Composable
fun ResponsiveTypography(): Typography {
    val configuration = LocalConfiguration.current
    val screenWidth = configuration.screenWidthDp.dp
    
    // Escalar tipografía según el ancho de pantalla
    val scale = when {
        screenWidth < 360.dp -> 0.9f
        screenWidth > 600.dp -> 1.1f
        else -> 1f
    }
    
    return Typography(
        displayLarge = TextStyle(
            fontSize = (57.sp * scale).value.sp,
            lineHeight = (64.sp * scale).value.sp
        ),
        headlineMedium = TextStyle(
            fontSize = (28.sp * scale).value.sp,
            lineHeight = (36.sp * scale).value.sp
        )
        // ... resto de estilos
    )
}
```

## Shapes Personalizadas

### 1. Shape System Extendido

```kotlin
// Shapes personalizadas
val AppShapes = Shapes(
    extraSmall = RoundedCornerShape(4.dp),
    small = RoundedCornerShape(8.dp),
    medium = RoundedCornerShape(12.dp),
    large = RoundedCornerShape(16.dp),
    extraLarge = RoundedCornerShape(28.dp)
)

// Shapes adicionales
object CustomShapes {
    val topRounded = RoundedCornerShape(
        topStart = 16.dp,
        topEnd = 16.dp,
        bottomStart = 0.dp,
        bottomEnd = 0.dp
    )
    
    val bottomRounded = RoundedCornerShape(
        topStart = 0.dp,
        topEnd = 0.dp,
        bottomStart = 16.dp,
        bottomEnd = 16.dp
    )
    
    val leftRounded = RoundedCornerShape(
        topStart = 16.dp,
        topEnd = 0.dp,
        bottomStart = 16.dp,
        bottomEnd = 0.dp
    )
    
    val rightRounded = RoundedCornerShape(
        topStart = 0.dp,
        topEnd = 16.dp,
        bottomStart = 0.dp,
        bottomEnd = 16.dp
    )
    
    // Cut corners (Material 2 style)
    val cutCorner = CutCornerShape(8.dp)
    
    // Asymmetric shapes
    val asymmetric = RoundedCornerShape(
        topStart = 24.dp,
        topEnd = 8.dp,
        bottomStart = 8.dp,
        bottomEnd = 24.dp
    )
}
```

### 2. Shapes Dinámicas

```kotlin
@Composable
fun DynamicShapes(isExpanded: Boolean): RoundedCornerShape {
    val animatedCorner by animateDpAsState(
        targetValue = if (isExpanded) 24.dp else 8.dp,
        label = "corner animation"
    )
    
    return RoundedCornerShape(animatedCorner)
}
```

## Dimensiones y Spacing

### 1. Sistema de Spacing

```kotlin
object AppDimensions {
    // Spacing scale
    val spacing0 = 0.dp
    val spacing1 = 4.dp
    val spacing2 = 8.dp
    val spacing3 = 12.dp
    val spacing4 = 16.dp
    val spacing5 = 20.dp
    val spacing6 = 24.dp
    val spacing8 = 32.dp
    val spacing10 = 40.dp
    val spacing12 = 48.dp
    val spacing16 = 64.dp
    
    // Component sizes
    val buttonHeight = 48.dp
    val iconSize = 24.dp
    val iconSizeSmall = 16.dp
    val iconSizeLarge = 32.dp
    
    // Layout
    val screenPadding = 16.dp
    val cardElevation = 4.dp
    val dividerThickness = 1.dp
    
    // Responsive
    @Composable
    fun contentMaxWidth(): Dp {
        val configuration = LocalConfiguration.current
        return when {
            configuration.screenWidthDp > 840 -> 840.dp
            else -> Dp.Infinity
        }
    }
}

// CompositionLocal para dimensiones
val LocalDimensions = staticCompositionLocalOf { AppDimensions }

@Composable
fun ProvideAppDimensions(content: @Composable () -> Unit) {
    CompositionLocalProvider(LocalDimensions provides AppDimensions) {
        content()
    }
}

// Extension
val MaterialTheme.dimensions: AppDimensions
    @Composable
    @ReadOnlyComposable
    get() = LocalDimensions.current
```

## Theme Switcher

### 1. Theme Manager con Estado

```kotlin
enum class ThemeMode {
    LIGHT, DARK, SYSTEM
}

class ThemeManager {
    private val _themeMode = MutableStateFlow(ThemeMode.SYSTEM)
    val themeMode: StateFlow<ThemeMode> = _themeMode.asStateFlow()
    
    private val _useDynamicColors = MutableStateFlow(true)
    val useDynamicColors: StateFlow<Boolean> = _useDynamicColors.asStateFlow()
    
    fun setThemeMode(mode: ThemeMode) {
        _themeMode.value = mode
    }
    
    fun toggleDynamicColors() {
        _useDynamicColors.value = !_useDynamicColors.value
    }
}

@Composable
fun AppWithThemeManager(
    themeManager: ThemeManager,
    content: @Composable () -> Unit
) {
    val themeMode by themeManager.themeMode.collectAsState()
    val useDynamicColors by themeManager.useDynamicColors.collectAsState()
    val systemInDarkTheme = isSystemInDarkTheme()
    
    val darkTheme = when (themeMode) {
        ThemeMode.LIGHT -> false
        ThemeMode.DARK -> true
        ThemeMode.SYSTEM -> systemInDarkTheme
    }
    
    MyAppTheme(
        darkTheme = darkTheme,
        dynamicColor = useDynamicColors,
        content = content
    )
}
```

### 2. Theme Switcher UI

```kotlin
@Composable
fun ThemeSettingsScreen(themeManager: ThemeManager) {
    val themeMode by themeManager.themeMode.collectAsState()
    val useDynamicColors by themeManager.useDynamicColors.collectAsState()
    
    Column(Modifier.padding(16.dp)) {
        Text("Theme Settings", style = MaterialTheme.typography.headlineMedium)
        
        Spacer(Modifier.height(16.dp))
        
        // Theme Mode Selection
        Text("Theme Mode", style = MaterialTheme.typography.titleMedium)
        
        ThemeMode.values().forEach { mode ->
            Row(
                verticalAlignment = Alignment.CenterVertically,
                modifier = Modifier
                    .fillMaxWidth()
                    .clickable { themeManager.setThemeMode(mode) }
                    .padding(vertical = 8.dp)
            ) {
                RadioButton(
                    selected = themeMode == mode,
                    onClick = { themeManager.setThemeMode(mode) }
                )
                Spacer(Modifier.width(8.dp))
                Text(
                    when (mode) {
                        ThemeMode.LIGHT -> "Light"
                        ThemeMode.DARK -> "Dark"
                        ThemeMode.SYSTEM -> "System Default"
                    }
                )
            }
        }
        
        Spacer(Modifier.height(16.dp))
        
        // Dynamic Colors Toggle (Android 12+)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
            Row(
                verticalAlignment = Alignment.CenterVertically,
                horizontalArrangement = Arrangement.SpaceBetween,
                modifier = Modifier.fillMaxWidth()
            ) {
                Text("Dynamic Colors")
                Switch(
                    checked = useDynamicColors,
                    onCheckedChange = { themeManager.toggleDynamicColors() }
                )
            }
        }
    }
}
```

## Multi-Brand Theming

### 1. Soporte para Múltiples Marcas

```kotlin
sealed class Brand {
    object Primary : Brand()
    object Secondary : Brand()
    object Partner : Brand()
}

object BrandThemes {
    val primaryLight = lightColorScheme(
        primary = Color(0xFF6750A4),
        secondary = Color(0xFF625B71),
        tertiary = Color(0xFF7D5260)
    )
    
    val secondaryLight = lightColorScheme(
        primary = Color(0xFF0066CC),
        secondary = Color(0xFF00A86B),
        tertiary = Color(0xFFFF6B6B)
    )
    
    val partnerLight = lightColorScheme(
        primary = Color(0xFFFF5722),
        secondary = Color(0xFF607D8B),
        tertiary = Color(0xFF9C27B0)
    )
    
    fun getColorScheme(brand: Brand, darkTheme: Boolean): ColorScheme {
        return when (brand) {
            Brand.Primary -> if (darkTheme) primaryDark else primaryLight
            Brand.Secondary -> if (darkTheme) secondaryDark else secondaryLight
            Brand.Partner -> if (darkTheme) partnerDark else partnerLight
        }
    }
}

@Composable
fun BrandedTheme(
    brand: Brand,
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = BrandThemes.getColorScheme(brand, darkTheme)
    
    MaterialTheme(
        colorScheme = colorScheme,
        content = content
    )
}
```

## Custom Theme Attributes

### 1. Atributos Personalizados del Theme

```kotlin
// Elevation system
data class ElevationLevels(
    val level0: Dp = 0.dp,
    val level1: Dp = 1.dp,
    val level2: Dp = 3.dp,
    val level3: Dp = 6.dp,
    val level4: Dp = 8.dp,
    val level5: Dp = 12.dp
)

val LocalElevation = staticCompositionLocalOf { ElevationLevels() }

// Animation durations
data class AnimationDurations(
    val short: Int = 150,
    val medium: Int = 300,
    val long: Int = 500
)

val LocalAnimationDurations = staticCompositionLocalOf { AnimationDurations() }

// Border widths
data class BorderWidths(
    val thin: Dp = 1.dp,
    val medium: Dp = 2.dp,
    val thick: Dp = 4.dp
)

val LocalBorderWidths = staticCompositionLocalOf { BorderWidths() }

// Compose all in theme
@Composable
fun CompleteAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    CompositionLocalProvider(
        LocalElevation provides ElevationLevels(),
        LocalAnimationDurations provides AnimationDurations(),
        LocalBorderWidths provides BorderWidths(),
        LocalExtendedColors provides if (darkTheme) DarkExtendedColors else LightExtendedColors
    ) {
        MaterialTheme(
            colorScheme = if (darkTheme) DarkColorScheme else LightColorScheme,
            typography = AppTypography,
            shapes = AppShapes,
            content = content
        )
    }
}

// Extensions para acceso fácil
val MaterialTheme.elevation: ElevationLevels
    @Composable
    @ReadOnlyComposable
    get() = LocalElevation.current

val MaterialTheme.animationDurations: AnimationDurations
    @Composable
    @ReadOnlyComposable
    get() = LocalAnimationDurations.current

val MaterialTheme.borderWidths: BorderWidths
    @Composable
    @ReadOnlyComposable
    get() = LocalBorderWidths.current
```

## Theme Preview

### 1. Preview con Diferentes Themes

```kotlin
@Preview(name = "Light Theme", showBackground = true)
@Preview(name = "Dark Theme", uiMode = Configuration.UI_MODE_NIGHT_YES)
@Composable
fun ThemedPreview() {
    MyAppTheme {
        Surface {
            MyComponent()
        }
    }
}

// Preview con múltiples configuraciones
@Preview(name = "Phone", device = Devices.PHONE)
@Preview(name = "Tablet", device = Devices.TABLET)
@Preview(name = "Desktop", device = Devices.DESKTOP)
@Composable
fun ResponsivePreview() {
    MyAppTheme {
        ResponsiveLayout()
    }
}

// Preview con diferentes brands
@Preview(name = "Primary Brand")
@Composable
fun PrimaryBrandPreview() {
    BrandedTheme(brand = Brand.Primary) {
        MyComponent()
    }
}

@Preview(name = "Secondary Brand")
@Composable
fun SecondaryBrandPreview() {
    BrandedTheme(brand = Brand.Secondary) {
        MyComponent()
    }
}
```

## Theme Testing

### 1. Test de Themes

```kotlin
@Test
fun testLightTheme() {
    composeTestRule.setContent {
        MyAppTheme(darkTheme = false) {
            val colors = MaterialTheme.colorScheme
            assertEquals(LightColorScheme.primary, colors.primary)
        }
    }
}

@Test
fun testDarkTheme() {
    composeTestRule.setContent {
        MyAppTheme(darkTheme = true) {
            val colors = MaterialTheme.colorScheme
            assertEquals(DarkColorScheme.primary, colors.primary)
        }
    }
}

@Test
fun testExtendedColors() {
    composeTestRule.setContent {
        MyAppTheme {
            val extendedColors = MaterialTheme.extendedColors
            assertNotNull(extendedColors.success)
        }
    }
}
```

## Mejores Prácticas

### 1. Organización de Archivos

```
theme/
├── Color.kt              // Color schemes
├── Type.kt               // Typography
├── Shape.kt              // Shapes
├── Theme.kt              // MaterialTheme wrapper
├── Dimensions.kt         // Spacing and sizes
├── ExtendedColors.kt     // Custom colors
└── ThemeManager.kt       // Theme state management
```

### 2. Naming Conventions

```kotlin
// ✅ BUENO - Nombres descriptivos
val successColor = Color(0xFF4CAF50)
val errorContainerColor = Color(0xFFF9DEDC)

// ❌ MALO - Nombres genéricos
val green = Color(0xFF4CAF50)
val lightRed = Color(0xFFF9DEDC)
```

### 3. Evitar Hardcoded Colors

```kotlin
// ❌ MALO
Text("Hello", color = Color.Blue)

// ✅ BUENO
Text("Hello", color = MaterialTheme.colorScheme.primary)
```

## Recursos

- [Material Design 3 Theming](https://m3.material.io/styles/color/overview)
- [Compose Theming](https://developer.android.com/jetpack/compose/designsystems/material)
- [Dynamic Color](https://m3.material.io/styles/color/dynamic-color/overview)
- [Material Theme Builder](https://material-foundation.github.io/material-theme-builder/)

---
*Un buen sistema de theming hace que la app sea consistente y fácil de mantener*
