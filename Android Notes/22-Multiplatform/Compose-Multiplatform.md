# Compose Multiplatform

## Setup

```kotlin
plugins {
    kotlin("multiplatform")
    id("org.jetbrains.compose")
}

kotlin {
    androidTarget()
    jvm("desktop")
    
    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation(compose.runtime)
                implementation(compose.foundation)
                implementation(compose.material3)
            }
        }
    }
}
```

## Shared UI

```kotlin
@Composable
fun App() {
    MaterialTheme {
        Scaffold {
            Text("Compose Multiplatform!")
        }
    }
}

// Android
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent { App() }
    }
}

// Desktop
fun main() = application {
    Window(onCloseRequest = ::exitApplication) {
        App()
    }
}
```

## Recursos
- [Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/)
