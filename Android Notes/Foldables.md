# Foldables

## Window Manager

```kotlin
dependencies {
    implementation("androidx.window:window:1.2.0")
}
```

## Detectar Foldable

```kotlin
val windowInfoTracker = WindowInfoTracker.getOrCreate(this)

lifecycleScope.launch {
    windowInfoTracker.windowLayoutInfo(this@MainActivity)
        .collect { layoutInfo ->
            layoutInfo.displayFeatures.forEach { feature ->
                if (feature is FoldingFeature) {
                    when (feature.state) {
                        FoldingFeature.State.FLAT -> {}
                        FoldingFeature.State.HALF_OPENED -> {}
                    }
                }
            }
        }
}
```

## Recursos
- [Foldables](https://developer.android.com/guide/topics/large-screens/learn-about-foldables)
