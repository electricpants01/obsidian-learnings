# Kotlin Multiplatform (KMP)

## Setup

```kotlin
plugins {
    kotlin("multiplatform")
}

kotlin {
    androidTarget()
    iosX64()
    iosArm64()
    iosSimulatorArm64()
    
    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
            }
        }
        val androidMain by getting
        val iosMain by creating
    }
}
```

## Shared Code

```kotlin
// commonMain
expect class Platform() {
    val name: String
}

// androidMain
actual class Platform {
    actual val name: String = "Android ${android.os.Build.VERSION.SDK_INT}"
}

// iosMain
actual class Platform {
    actual val name: String = UIDevice.currentDevice.systemName()
}
```

## Recursos
- [KMP](https://kotlinlang.org/docs/multiplatform.html)
