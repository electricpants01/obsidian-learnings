# Performance Optimization

## LazyColumn Optimization

```kotlin
LazyColumn {
    items(
        items = items,
        key = { it.id } // Stable keys
    ) { item ->
        ItemCard(item)
    }
}
```

## Remember

```kotlin
@Composable
fun MyScreen() {
    val expensiveData = remember {
        computeExpensiveData()
    }
}
```

## Baseline Profiles

```kotlin
// Generate baseline profile
./gradlew :app:generateBaselineProfile
```

## Recursos
- [Performance](https://developer.android.com/topic/performance)
