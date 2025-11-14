# Code Quality

## Detekt

```kotlin
plugins {
    id("io.gitlab.arturbosch.detekt") version "1.23.4"
}

detekt {
    buildUponDefaultConfig = true
    config.setFrom("$projectDir/config/detekt.yml")
}
```

## KtLint

```kotlin
plugins {
    id("org.jlleitschuh.gradle.ktlint") version "11.6.1"
}
```

## SonarQube

```kotlin
plugins {
    id("org.sonarqube") version "4.4.1.3373"
}
```

## Recursos
- [Code Quality](https://developer.android.com/studio/write/lint)
