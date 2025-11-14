# Large Screens

## Window Size Classes

```kotlin
@Composable
fun AdaptiveLayout() {
    val windowSizeClass = calculateWindowSizeClass(this)
    
    when (windowSizeClass.widthSizeClass) {
        WindowWidthSizeClass.Compact -> {
            // Phone portrait
            CompactLayout()
        }
        WindowWidthSizeClass.Medium -> {
            // Tablet portrait, foldable
            MediumLayout()
        }
        WindowWidthSizeClass.Expanded -> {
            // Tablet landscape, desktop
            ExpandedLayout()
        }
    }
}
```

## Responsive UI

```kotlin
@Composable
fun ResponsiveContent(windowSize: WindowSizeClass) {
    if (windowSize.widthSizeClass == WindowWidthSizeClass.Expanded) {
        Row {
            NavigationRail()
            MainContent()
        }
    } else {
        Scaffold(bottomBar = { BottomNavigation() }) {
            MainContent()
        }
    }
}
```

## Recursos
- [Large Screens](https://developer.android.com/guide/topics/large-screens)
