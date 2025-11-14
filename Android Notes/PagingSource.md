# PagingSource

## Definición

Clase abstracta que define cómo cargar chunks de datos.

## Implementación

```kotlin
class UsersPagingSource(
    private val api: ApiService,
    private val query: String
) : PagingSource<Int, User>() {
    
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, User> {
        return try {
            val page = params.key ?: INITIAL_PAGE
            val response = api.searchUsers(
                query = query,
                page = page,
                perPage = params.loadSize
            )
            
            LoadResult.Page(
                data = response.users,
                prevKey = if (page == INITIAL_PAGE) null else page - 1,
                nextKey = if (response.users.isEmpty()) null else page + 1
            )
        } catch (e: IOException) {
            LoadResult.Error(e)
        } catch (e: HttpException) {
            LoadResult.Error(e)
        }
    }
    
    override fun getRefreshKey(state: PagingState<Int, User>): Int? {
        return state.anchorPosition?.let { anchorPosition ->
            val anchorPage = state.closestPageToPosition(anchorPosition)
            anchorPage?.prevKey?.plus(1) ?: anchorPage?.nextKey?.minus(1)
        }
    }
    
    companion object {
        private const val INITIAL_PAGE = 1
    }
}
```

## LoadParams

```kotlin
sealed class LoadParams<Key : Any> {
    abstract val key: Key?
    abstract val loadSize: Int
    
    // Refresh - carga inicial
    class Refresh<Key : Any>(
        override val key: Key?,
        override val loadSize: Int
    ) : LoadParams<Key>()
    
    // Prepend - cargar hacia atrás
    class Prepend<Key : Any>(
        override val key: Key,
        override val loadSize: Int
    ) : LoadParams<Key>()
    
    // Append - cargar hacia adelante
    class Append<Key : Any>(
        override val key: Key,
        override val loadSize: Int
    ) : LoadParams<Key>()
}
```

## LoadResult

```kotlin
sealed class LoadResult<Key : Any, Value : Any> {
    // Success
    data class Page<Key : Any, Value : Any>(
        val data: List<Value>,
        val prevKey: Key?,
        val nextKey: Key?
    ) : LoadResult<Key, Value>()
    
    // Error
    data class Error<Key : Any, Value : Any>(
        val throwable: Throwable
    ) : LoadResult<Key, Value>()
    
    // Invalid
    class Invalid<Key : Any, Value : Any> : LoadResult<Key, Value>()
}
```

## String Keys

```kotlin
class ArticlesPagingSource(
    private val api: ApiService
) : PagingSource<String, Article>() {
    
    override suspend fun load(params: LoadParams<String>): LoadResult<String, Article> {
        return try {
            val nextToken = params.key
            val response = api.getArticles(
                nextToken = nextToken,
                limit = params.loadSize
            )
            
            LoadResult.Page(
                data = response.articles,
                prevKey = null, // Solo paginación hacia adelante
                nextKey = response.nextToken
            )
        } catch (e: Exception) {
            LoadResult.Error(e)
        }
    }
    
    override fun getRefreshKey(state: PagingState<String, Article>): String? = null
}
```

## Invalidación

```kotlin
class MyPagingSource : PagingSource<Int, Item>() {
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Item> {
        // Load data
    }
    
    override fun getRefreshKey(state: PagingState<Int, Item>): Int? = null
    
    fun invalidate() {
        invalidate() // Trigger refresh
    }
}

// En Repository
class Repository {
    private var currentSource: MyPagingSource? = null
    
    fun getItems(): Flow<PagingData<Item>> = Pager(
        config = PagingConfig(pageSize = 20),
        pagingSourceFactory = {
            MyPagingSource().also { currentSource = it }
        }
    ).flow
    
    fun refresh() {
        currentSource?.invalidate()
    }
}
```

## Recursos
- [PagingSource](https://developer.android.com/reference/kotlin/androidx/paging/PagingSource)
