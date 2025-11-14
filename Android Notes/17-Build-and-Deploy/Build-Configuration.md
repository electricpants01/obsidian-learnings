# Build Configuration

## Build Types

```kotlin
android {
    buildTypes {
        debug {
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-DEBUG"
            isDebuggable = true
        }
        release {
            isMinifyEnabled = true
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
        }
    }
}
```

## Product Flavors

```kotlin
flavorDimensions += "version"
productFlavors {
    create("free") {
        dimension = "version"
        applicationIdSuffix = ".free"
    }
    create("paid") {
        dimension = "version"
        applicationIdSuffix = ".paid"
    }
}
```

## Build Variants

```kotlin
// Combinaciones: freeDebug, freeRelease, paidDebug, paidRelease
```

## Recursos
- [Build Config](https://developer.android.com/studio/build)
