# Paging with Room

## PagingSource desde Room

```kotlin
@Dao
interface ItemDao {
    @Query("SELECT * FROM items ORDER BY id ASC")
    fun pagingSource(): PagingSource<Int, ItemEntity>
}

// Repository
class ItemsRepository(private val dao: ItemDao) {
    fun getItems(): Flow<PagingData<ItemEntity>> = Pager(
        config = PagingConfig(pageSize = 20),
        pagingSourceFactory = { dao.pagingSource() }
    ).flow
}
```

## Con RemoteMediator

```kotlin
@OptIn(ExperimentalPagingApi::class)
fun getItems(): Flow<PagingData<ItemEntity>> = Pager(
    config = PagingConfig(pageSize = 20),
    remoteMediator = ItemsRemoteMediator(database, api),
    pagingSourceFactory = { database.itemDao().pagingSource() }
).flow
```

## Recursos
- [Paging with Room](https://developer.android.com/topic/libraries/architecture/paging/v3-network-db)
