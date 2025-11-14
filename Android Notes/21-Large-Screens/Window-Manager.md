# Window Manager

## Window Metrics

```kotlin
dependencies {
    implementation("androidx.window:window:1.2.0")
}
```

## Window Size

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        lifecycleScope.launch {
            lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
                WindowInfoTracker.getOrCreate(this@MainActivity)
                    .windowMetricsFlow()
                    .collect { windowMetrics ->
                        val width = windowMetrics.bounds.width()
                        val height = windowMetrics.bounds.height()
                    }
            }
        }
    }
}
```

## Recursos
- [Window Manager](https://developer.android.com/jetpack/androidx/releases/window)
