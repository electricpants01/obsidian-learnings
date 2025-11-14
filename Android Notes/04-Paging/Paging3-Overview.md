# Paging 3 Overview

## ¿Qué es Paging 3?

Librería de Jetpack para cargar datos de forma gradual (paginada) de manera eficiente.

## Setup

```kotlin
dependencies {
    implementation("androidx.paging:paging-runtime:3.2.1")
    implementation("androidx.paging:paging-compose:3.2.1")
}
```

## Componentes Principales

1. **PagingSource**: Define cómo obtener datos
2. **Pager**: Crea Flow de PagingData
3. **PagingData**: Contiene datos paginados
4. **RemoteMediator**: Cache + Network
5. **LazyPagingItems**: Compose integration

## Ejemplo Básico

```kotlin
// PagingSource
class ItemsPagingSource(
    private val api: ApiService
) : PagingSource<Int, Item>() {
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Item> {
        return try {
            val page = params.key ?: 1
            val response = api.getItems(page, params.loadSize)
            
            LoadResult.Page(
                data = response.items,
                prevKey = if (page == 1) null else page - 1,
                nextKey = if (response.items.isEmpty()) null else page + 1
            )
        } catch (e: Exception) {
            LoadResult.Error(e)
        }
    }
    
    override fun getRefreshKey(state: PagingState<Int, Item>): Int? {
        return state.anchorPosition?.let { position ->
            state.closestPageToPosition(position)?.prevKey?.plus(1)
                ?: state.closestPageToPosition(position)?.nextKey?.minus(1)
        }
    }
}

// Repository
class ItemsRepository(private val api: ApiService) {
    fun getItems(): Flow<PagingData<Item>> {
        return Pager(
            config = PagingConfig(
                pageSize = 20,
                enablePlaceholders = false
            ),
            pagingSourceFactory = { ItemsPagingSource(api) }
        ).flow
    }
}

// ViewModel
class ItemsViewModel(repository: ItemsRepository) : ViewModel() {
    val items: Flow<PagingData<Item>> = repository.getItems()
        .cachedIn(viewModelScope)
}

// Compose
@Composable
fun ItemsScreen(viewModel: ItemsViewModel = viewModel()) {
    val items = viewModel.items.collectAsLazyPagingItems()
    
    LazyColumn {
        items(items.itemCount) { index ->
            items[index]?.let { item ->
                ItemCard(item)
            }
        }
        
        item {
            when (items.loadState.append) {
                is LoadState.Loading -> LoadingIndicator()
                is LoadState.Error -> ErrorMessage()
                else -> {}
            }
        }
    }
}
```

## Recursos
- [Paging 3 Documentation](https://developer.android.com/topic/libraries/architecture/paging/v3-overview)
