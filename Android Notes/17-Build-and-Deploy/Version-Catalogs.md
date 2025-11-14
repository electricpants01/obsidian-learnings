# Version Catalogs

## libs.versions.toml

```toml
[versions]
kotlin = "1.9.21"
compose = "1.6.0"
androidx-core = "1.12.0"

[libraries]
androidx-core-ktx = { module = "androidx.core:core-ktx", version.ref = "androidx-core" }
compose-ui = { module = "androidx.compose.ui:ui", version.ref = "compose" }
compose-material3 = { module = "androidx.compose.material3:material3", version.ref = "compose" }

[plugins]
android-application = { id = "com.android.application", version = "8.2.0" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }

[bundles]
compose = ["compose-ui", "compose-material3"]
```

## build.gradle.kts

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
}

dependencies {
    implementation(libs.androidx.core.ktx)
    implementation(libs.bundles.compose)
}
```

## Recursos
- [Version Catalogs](https://developer.android.com/build/migrate-to-catalogs)
