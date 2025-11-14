# Modularization

## Estructura

```
app/
  - UI layer, Activities, Application class
core/
  - common/ - Utilities compartidos
  - data/ - Repositorios, Data sources
  - domain/ - Use cases, Models
  - network/ - Retrofit, API
  - database/ - Room
feature/
  - home/ - Feature module
  - profile/ - Feature module
  - settings/ - Feature module
```

## Gradle Module

```kotlin
// settings.gradle.kts
include(":app")
include(":core:common")
include(":core:data")
include(":feature:home")

// feature/home/build.gradle.kts
dependencies {
    implementation(project(":core:common"))
    implementation(project(":core:data"))
}
```

## Recursos
- [Modularization](https://developer.android.com/topic/modularization)
