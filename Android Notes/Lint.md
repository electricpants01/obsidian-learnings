# Lint

## Configuración

```kotlin
android {
    lint {
        baseline = file("lint-baseline.xml")
        checkReleaseBuilds = true
        abortOnError = false
        warningsAsErrors = true
    }
}
```

## Suppress Warnings

```kotlin
@SuppressLint("UnusedMaterial3ScaffoldPaddingParameter")
@Composable
fun MyScreen() {
    Scaffold { }
}
```

## Custom Rules

```kotlin
// lint.xml
<lint>
    <issue id="IconMissingDensityFolder" severity="ignore" />
</lint>
```

## Recursos
- [Lint](https://developer.android.com/studio/write/lint)
