# LoadState en Paging

## LoadState

Representa el estado de carga de la paginación.

```kotlin
sealed class LoadState {
    object NotLoading : LoadState()
    object Loading : LoadState()
    data class Error(val error: Throwable) : LoadState()
}
```

## CombinedLoadStates

```kotlin
data class CombinedLoadStates(
    val refresh: LoadState,  // Estado de refresh
    val prepend: LoadState,  // Estado de carga hacia atrás
    val append: LoadState,   // Estado de carga hacia adelante
    val source: LoadStates,  // Estados de PagingSource
    val mediator: LoadStates? // Estados de RemoteMediator
)
```

## En Compose

```kotlin
@Composable
fun ItemsList(viewModel: MyViewModel = viewModel()) {
    val items = viewModel.items.collectAsLazyPagingItems()
    
    LazyColumn {
        // Refresh state
        when (items.loadState.refresh) {
            is LoadState.Loading -> {
                item { LoadingIndicator() }
            }
            is LoadState.Error -> {
                val error = (items.loadState.refresh as LoadState.Error).error
                item { ErrorMessage(error.message) }
            }
            else -> {
                items(items.itemCount) { index ->
                    items[index]?.let { item ->
                        ItemCard(item)
                    }
                }
            }
        }
        
        // Append state
        when (items.loadState.append) {
            is LoadState.Loading -> {
                item { LoadingIndicator() }
            }
            is LoadState.Error -> {
                val error = (items.loadState.append as LoadState.Error).error
                item { 
                    RetryButton { items.retry() }
                }
            }
            else -> {}
        }
    }
}
```

## Listener

```kotlin
@Composable
fun ItemsList() {
    val items = viewModel.items.collectAsLazyPagingItems()
    
    LaunchedEffect(items.loadState) {
        val errorState = items.loadState.refresh as? LoadState.Error
            ?: items.loadState.append as? LoadState.Error
            ?: items.loadState.prepend as? LoadState.Error
        
        errorState?.let {
            // Show error
        }
    }
}
```

## Recursos
- [LoadState](https://developer.android.com/reference/kotlin/androidx/paging/LoadState)
