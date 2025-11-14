# Wear OS

## Setup

```kotlin
dependencies {
    implementation("androidx.wear:wear:1.3.0")
}
```

## Wear Activity

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            WearApp()
        }
    }
}

@Composable
fun WearApp() {
    MaterialTheme {
        Scaffold(
            timeText = { TimeText() }
        ) {
            ScalingLazyColumn {
                item {
                    Text("Wear OS App")
                }
            }
        }
    }
}
```

## Recursos
- [Wear OS](https://developer.android.com/training/wearables)
