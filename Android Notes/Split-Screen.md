# Split Screen

## Multi-Window

```xml
<activity
    android:name=".MainActivity"
    android:resizeableActivity="true" />
```

## Detectar Multi-Window

```kotlin
override fun onMultiWindowModeChanged(
    isInMultiWindowMode: Boolean,
    newConfig: Configuration
) {
    if (isInMultiWindowMode) {
        // App en split screen
    } else {
        // App en pantalla completa
    }
}
```

## Compose

```kotlin
@Composable
fun AdaptToMultiWindow() {
    val configuration = LocalConfiguration.current
    val isInMultiWindow = remember(configuration) {
        configuration.screenWidthDp < 600
    }
    
    if (isInMultiWindow) {
        CompactLayout()
    } else {
        ExpandedLayout()
    }
}
```

## Recursos
- [Multi-Window](https://developer.android.com/guide/topics/large-screens/multi-window-support)
