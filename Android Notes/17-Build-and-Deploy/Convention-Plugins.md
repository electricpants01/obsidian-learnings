# Convention Plugins

## buildSrc/build.gradle.kts

```kotlin
plugins {
    `kotlin-dsl`
}

repositories {
    google()
    mavenCentral()
}

dependencies {
    implementation("com.android.tools.build:gradle:8.2.0")
    implementation("org.jetbrains.kotlin:kotlin-gradle-plugin:1.9.21")
}
```

## buildSrc/src/main/kotlin/AndroidLibraryConventionPlugin.kt

```kotlin
import com.android.build.gradle.LibraryExtension
import org.gradle.api.Plugin
import org.gradle.api.Project

class AndroidLibraryConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply("com.android.library")
                apply("org.jetbrains.kotlin.android")
            }
            
            extensions.configure<LibraryExtension>("android") {
                compileSdk = 34
                defaultConfig {
                    minSdk = 24
                }
                compileOptions {
                    sourceCompatibility = JavaVersion.VERSION_17
                    targetCompatibility = JavaVersion.VERSION_17
                }
            }
        }
    }
}
```

## Uso

```kotlin
plugins {
    id("my.android.library")
}
```

## Recursos
- [Convention Plugins](https://developer.android.com/build/migrate-to-catalogs#migrate-build-logic)
