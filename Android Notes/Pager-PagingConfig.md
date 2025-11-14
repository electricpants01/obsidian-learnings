# Pager y PagingConfig

## Pager

Crea Flow<PagingData> desde PagingSource.

```kotlin
val pagingData: Flow<PagingData<Item>> = Pager(
    config = PagingConfig(
        pageSize = 20,
        prefetchDistance = 5,
        enablePlaceholders = false,
        initialLoadSize = 40,
        maxSize = 200
    ),
    pagingSourceFactory = { ItemsPagingSource(api) }
).flow
```

## PagingConfig

```kotlin
data class PagingConfig(
    val pageSize: Int,                    // Items por página
    val prefetchDistance: Int = pageSize, // Cuándo cargar siguiente
    val enablePlaceholders: Boolean = true, // Mostrar placeholders
    val initialLoadSize: Int = pageSize * 3, // Primera carga
    val maxSize: Int = MAX_SIZE_UNBOUNDED,   // Tamaño máximo
    val jumpThreshold: Int = COUNT_UNDEFINED // Para saltos grandes
)
```

## Parámetros

```kotlin
// pageSize - items por página
PagingConfig(pageSize = 20)

// prefetchDistance - cargar antes de llegar al final
PagingConfig(
    pageSize = 20,
    prefetchDistance = 10 // Carga 10 items antes del final
)

// enablePlaceholders - mostrar null items
PagingConfig(
    pageSize = 20,
    enablePlaceholders = true // Muestra placeholders
)

// initialLoadSize - primera carga
PagingConfig(
    pageSize = 20,
    initialLoadSize = 60 // Carga 3 páginas inicialmente
)

// maxSize - límite de items en memoria
PagingConfig(
    pageSize = 20,
    maxSize = 200 // Max 200 items en memoria
)
```

## cachedIn

```kotlin
class MyViewModel(repository: Repository) : ViewModel() {
    val items: Flow<PagingData<Item>> = repository.getItems()
        .cachedIn(viewModelScope) // Cache durante config changes
}
```

## Recursos
- [Pager](https://developer.android.com/reference/kotlin/androidx/paging/Pager)
- [PagingConfig](https://developer.android.com/reference/kotlin/androidx/paging/PagingConfig)
