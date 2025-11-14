# Adaptive Layouts

## Material3 Adaptive

```kotlin
dependencies {
    implementation("androidx.compose.material3:material3-adaptive:1.0.0-alpha05")
}
```

## NavigationSuiteScaffold

```kotlin
@Composable
fun AdaptiveApp() {
    val layoutType = calculateStandardNavigationType()
    
    NavigationSuiteScaffold(
        navigationSuiteItems = {
            items.forEach { item ->
                item(
                    icon = { Icon(item.icon, contentDescription = null) },
                    label = { Text(item.label) },
                    selected = selectedItem == item,
                    onClick = { selectedItem = item }
                )
            }
        }
    ) {
        // Content
    }
}
```

## ListDetailPaneScaffold

```kotlin
@Composable
fun ListDetail() {
    ListDetailPaneScaffold(
        listPane = { ListPane() },
        detailPane = { DetailPane() }
    )
}
```

## Recursos
- [Adaptive Layouts](https://developer.android.com/develop/ui/compose/layouts/adaptive)
