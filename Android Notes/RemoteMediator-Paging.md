# RemoteMediator

## ¿Qué es RemoteMediator?

Componente que coordina la carga desde red y caché (Room) para Paging 3.

## Implementación

```kotlin
@OptIn(ExperimentalPagingApi::class)
class ItemsRemoteMediator(
    private val database: AppDatabase,
    private val api: ApiService
) : RemoteMediator<Int, ItemEntity>() {
    
    override suspend fun load(
        loadType: LoadType,
        state: PagingState<Int, ItemEntity>
    ): MediatorResult {
        return try {
            val page = when (loadType) {
                LoadType.REFRESH -> 1
                LoadType.PREPEND -> return MediatorResult.Success(endOfPaginationReached = true)
                LoadType.APPEND -> {
                    val lastItem = state.lastItemOrNull()
                        ?: return MediatorResult.Success(endOfPaginationReached = true)
                    (lastItem.id / state.config.pageSize) + 1
                }
            }
            
            val response = api.getItems(page, state.config.pageSize)
            
            database.withTransaction {
                if (loadType == LoadType.REFRESH) {
                    database.itemDao().clearAll()
                }
                database.itemDao().insertAll(response.items)
            }
            
            MediatorResult.Success(
                endOfPaginationReached = response.items.isEmpty()
            )
        } catch (e: Exception) {
            MediatorResult.Error(e)
        }
    }
}
```

## Con Pager

```kotlin
@OptIn(ExperimentalPagingApi::class)
class ItemsRepository(
    private val database: AppDatabase,
    private val api: ApiService
) {
    fun getItems(): Flow<PagingData<ItemEntity>> {
        return Pager(
            config = PagingConfig(pageSize = 20),
            remoteMediator = ItemsRemoteMediator(database, api),
            pagingSourceFactory = { database.itemDao().pagingSource() }
        ).flow
    }
}
```

## LoadType

```kotlin
enum class LoadType {
    REFRESH,  // Carga inicial o refresh
    PREPEND,  // Cargar hacia atrás
    APPEND    // Cargar hacia adelante
}
```

## Recursos
- [RemoteMediator](https://developer.android.com/reference/kotlin/androidx/paging/RemoteMediator)
