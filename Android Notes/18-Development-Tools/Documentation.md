# Documentation

## KDoc

```kotlin
/**
 * Calcula el total de una lista de precios.
 *
 * @param prices Lista de precios
 * @return Total sumado
 * @throws IllegalArgumentException si la lista está vacía
 */
fun calculateTotal(prices: List<Double>): Double {
    require(prices.isNotEmpty()) { "Lista no puede estar vacía" }
    return prices.sum()
}
```

## Dokka

```kotlin
plugins {
    id("org.jetbrains.dokka") version "1.9.10"
}

tasks.dokkaHtml.configure {
    outputDirectory.set(buildDir.resolve("dokka"))
}
```

## README

```markdown
# My Android App

## Features
- Feature 1
- Feature 2

## Setup
1. Clone repo
2. Open in Android Studio
3. Run
```

## Recursos
- [Documentation](https://kotlinlang.org/docs/kotlin-doc.html)
