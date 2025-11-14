# Multi-Module

## Ventajas

- Build times más rápidos (builds paralelos)
- Separación de responsabilidades
- Reutilización de código
- Encapsulación mejorada

## Tipos de Módulos

```kotlin
// App module
android {
    namespace = "com.example.app"
}

// Android Library module
plugins {
    id("com.android.library")
}

// Pure Kotlin module
plugins {
    id("java-library")
    kotlin("jvm")
}
```

## Dependency Graph

```
app
 ├─ feature:home
 │   └─ core:domain
 ├─ feature:profile
 │   └─ core:domain
 └─ core:data
     └─ core:network
```

## Recursos
- [Multi-module](https://developer.android.com/topic/modularization)
