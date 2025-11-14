# Gradle Basics

## build.gradle.kts

```kotlin
plugins {
    id("com.android.application")
    kotlin("android")
}

android {
    namespace = "com.example.app"
    compileSdk = 34
    
    defaultConfig {
        applicationId = "com.example.app"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }
}

dependencies {
    implementation("androidx.core:core-ktx:1.12.0")
}
```

## settings.gradle.kts

```kotlin
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "MyApp"
include(":app")
```

## Recursos
- [Gradle](https://developer.android.com/studio/build)
