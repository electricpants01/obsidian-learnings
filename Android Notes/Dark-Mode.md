# Dark Mode

## Implementar Dark Mode

```kotlin
// En Material3
MaterialTheme(
    colorScheme = if (isSystemInDarkTheme()) darkColorScheme() else lightColorScheme()
) {
    // Content
}
```

## Forzar Modo

```kotlin
// Forzar modo
AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES) // Dark
AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_NO) // Light
AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_FOLLOW_SYSTEM) // System
```

## Recursos por Modo

```xml
<!-- res/values/colors.xml -->
<color name="background">#FFFFFF</color>

<!-- res/values-night/colors.xml -->
<color name="background">#000000</color>
```

## Recursos
- [Dark Theme](https://developer.android.com/guide/topics/ui/look-and-feel/darktheme)
