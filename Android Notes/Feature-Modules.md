# Feature Modules

## Dynamic Feature

```kotlin
// build.gradle.kts (feature module)
plugins {
    id("com.android.dynamic-feature")
}

android {
    namespace = "com.example.feature"
}

dependencies {
    implementation(project(":app"))
}
```

## Instalar Feature

```kotlin
val request = SplitInstallRequest.newBuilder()
    .addModule("feature_name")
    .build()

splitInstallManager.startInstall(request)
    .addOnSuccessListener { sessionId ->
        // Feature installed
    }
```

## Recursos
- [Dynamic Features](https://developer.android.com/guide/playcore/feature-delivery)
