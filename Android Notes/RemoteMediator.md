# RemoteMediator - Sincronización Local/Remoto

## Descripción

RemoteMediator es un componente de Paging 3 que actúa como intermediario entre la red y la base de datos local. Permite implementar el patrón offline-first, sincronizando datos remotos con el almacenamiento local.

### Concepto Principal

**Database como fuente de verdad + Network como origen de datos**

RemoteMediator carga datos de la red y los almacena en la base de datos local, que actúa como la única fuente de verdad para la UI.

### Ventajas

- Soporte offline automático
- Caché local persistente
- Sincronización eficiente
- Paginación con datos locales y remotos
- Manejo de errores robusto

## Implementación Básica

```kotlin
@OptIn(ExperimentalPagingApi::class)
class UserRemoteMediator(
    private val database: AppDatabase,
    private val api: UserApi
) : RemoteMediator<Int, UserEntity>() {
    
    override suspend fun load(
        loadType: LoadType,
        state: PagingState<Int, UserEntity>
    ): MediatorResult {
        return try {
            // Determinar la página a cargar
            val page = when (loadType) {
                LoadType.REFRESH -> 0
                LoadType.PREPEND -> return MediatorResult.Success(
                    endOfPaginationReached = true
                )
                LoadType.APPEND -> {
                    val lastItem = state.lastItemOrNull()
                        ?: return MediatorResult.Success(
                            endOfPaginationReached = true
                        )
                    
                    // Calcular siguiente página basado en el último item
                    (lastItem.id / state.config.pageSize) + 1
                }
            }
            
            // Cargar datos de la API
            val users = api.getUsers(
                page = page,
                size = state.config.pageSize
            )
            
            // Guardar en base de datos
            database.withTransaction {
                if (loadType == LoadType.REFRESH) {
                    // Limpiar datos antiguos en refresh
                    database.userDao().clearAll()
                }
                
                database.userDao().insertAll(users.map { it.toEntity() })
            }
            
            MediatorResult.Success(
                endOfPaginationReached = users.isEmpty()
            )
        } catch (e: Exception) {
            MediatorResult.Error(e)
        }
    }
}
```

## Uso con Pager

```kotlin
class UserRepository(
    private val database: AppDatabase,
    private val api: UserApi
) {
    
    @OptIn(ExperimentalPagingApi::class)
    fun getUsersPaged(): Flow<PagingData<User>> {
        return Pager(
            config = PagingConfig(
                pageSize = 20,
                enablePlaceholders = false,
                prefetchDistance = 5
            ),
            remoteMediator = UserRemoteMediator(database, api),
            pagingSourceFactory = {
                // La base de datos es la fuente de verdad
                database.userDao().pagingSource()
            }
        ).flow.map { pagingData ->
            pagingData.map { it.toDomain() }
        }
    }
}

// En ViewModel
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {
    
    val users: Flow<PagingData<User>> = repository
        .getUsersPaged()
        .cachedIn(viewModelScope)
}

// En Compose
@Composable
fun UserListScreen(viewModel: UserViewModel = hiltViewModel()) {
    val users = viewModel.users.collectAsLazyPagingItems()
    
    LazyColumn {
        items(
            count = users.itemCount,
            key = { index -> users[index]?.id ?: index }
        ) { index ->
            users[index]?.let { user ->
                UserItem(user)
            }
        }
        
        users.apply {
            when {
                loadState.refresh is LoadState.Loading -> {
                    item { LoadingItem() }
                }
                loadState.append is LoadState.Loading -> {
                    item { LoadingItem() }
                }
                loadState.refresh is LoadState.Error -> {
                    item { ErrorItem(loadState.refresh as LoadState.Error) }
                }
            }
        }
    }
}
```

## RemoteMediator con Remote Keys

```kotlin
// Entidad para almacenar claves remotas
@Entity(tableName = "user_remote_keys")
data class UserRemoteKeys(
    @PrimaryKey
    val userId: String,
    val prevKey: Int?,
    val nextKey: Int?
)

@OptIn(ExperimentalPagingApi::class)
class UserRemoteMediator(
    private val database: AppDatabase,
    private val api: UserApi
) : RemoteMediator<Int, UserEntity>() {
    
    private val userDao = database.userDao()
    private val remoteKeysDao = database.userRemoteKeysDao()
    
    override suspend fun load(
        loadType: LoadType,
        state: PagingState<Int, UserEntity>
    ): MediatorResult {
        return try {
            val page = when (loadType) {
                LoadType.REFRESH -> {
                    val remoteKeys = getRemoteKeyClosestToCurrentPosition(state)
                    remoteKeys?.nextKey?.minus(1) ?: 0
                }
                LoadType.PREPEND -> {
                    val remoteKeys = getRemoteKeyForFirstItem(state)
                    val prevKey = remoteKeys?.prevKey
                        ?: return MediatorResult.Success(
                            endOfPaginationReached = remoteKeys != null
                        )
                    prevKey
                }
                LoadType.APPEND -> {
                    val remoteKeys = getRemoteKeyForLastItem(state)
                    val nextKey = remoteKeys?.nextKey
                        ?: return MediatorResult.Success(
                            endOfPaginationReached = remoteKeys != null
                        )
                    nextKey
                }
            }
            
            val response = api.getUsers(page = page, size = state.config.pageSize)
            val endOfPaginationReached = response.isEmpty()
            
            database.withTransaction {
                if (loadType == LoadType.REFRESH) {
                    userDao.clearAll()
                    remoteKeysDao.clearAll()
                }
                
                val prevKey = if (page == 0) null else page - 1
                val nextKey = if (endOfPaginationReached) null else page + 1
                
                val keys = response.map {
                    UserRemoteKeys(
                        userId = it.id,
                        prevKey = prevKey,
                        nextKey = nextKey
                    )
                }
                
                remoteKeysDao.insertAll(keys)
                userDao.insertAll(response.map { it.toEntity() })
            }
            
            MediatorResult.Success(endOfPaginationReached = endOfPaginationReached)
            
        } catch (e: Exception) {
            MediatorResult.Error(e)
        }
    }
    
    private suspend fun getRemoteKeyForLastItem(
        state: PagingState<Int, UserEntity>
    ): UserRemoteKeys? {
        return state.pages.lastOrNull { it.data.isNotEmpty() }?.data?.lastOrNull()
            ?.let { user ->
                remoteKeysDao.getRemoteKeys(user.id)
            }
    }
    
    private suspend fun getRemoteKeyForFirstItem(
        state: PagingState<Int, UserEntity>
    ): UserRemoteKeys? {
        return state.pages.firstOrNull { it.data.isNotEmpty() }?.data?.firstOrNull()
            ?.let { user ->
                remoteKeysDao.getRemoteKeys(user.id)
            }
    }
    
    private suspend fun getRemoteKeyClosestToCurrentPosition(
        state: PagingState<Int, UserEntity>
    ): UserRemoteKeys? {
        return state.anchorPosition?.let { position ->
            state.closestItemToPosition(position)?.id?.let { userId ->
                remoteKeysDao.getRemoteKeys(userId)
            }
        }
    }
}
```

## Documentación Oficial

- [Paging 3 with Network and Database](https://developer.android.com/topic/libraries/architecture/paging/v3-network-db)
- [RemoteMediator API](https://developer.android.com/reference/kotlin/androidx/paging/RemoteMediator)
- [Paging 3 Overview](https://developer.android.com/topic/libraries/architecture/paging/v3-overview)

## Videos Recomendados

- [Paging 3 Network and Database](https://www.youtube.com/watch?v=ZARz0pjm5YM) - Android Developers
- [RemoteMediator Deep Dive](https://www.youtube.com/watch?v=C0H54K63wNw) - Philipp Lackner
- [Offline-First with Paging 3](https://www.youtube.com/watch?v=P_jdHxVKbZQ) - Coding with Mitch

## Mejores Prácticas

1. **Database como SSOT**: Siempre usar la DB como fuente de verdad
2. **Remote Keys**: Usar tabla separada para claves de paginación
3. **Transactions**: Usar withTransaction para operaciones atómicas
4. **Error Handling**: Manejar errores de red apropiadamente
5. **Refresh Strategy**: Implementar refresh pull-to-refresh

## Patrón Completo con Refresh

```kotlin
@Composable
fun UserListScreen(viewModel: UserViewModel = hiltViewModel()) {
    val users = viewModel.users.collectAsLazyPagingItems()
    val pullRefreshState = rememberPullRefreshState(
        refreshing = users.loadState.refresh is LoadState.Loading,
        onRefresh = { users.refresh() }
    )
    
    Box(Modifier.pullRefresh(pullRefreshState)) {
        LazyColumn {
            items(users.itemCount) { index ->
                users[index]?.let { user ->
                    UserItem(user)
                }
            }
        }
        
        PullRefreshIndicator(
            refreshing = users.loadState.refresh is LoadState.Loading,
            state = pullRefreshState,
            modifier = Modifier.align(Alignment.TopCenter)
        )
    }
}
```

## Manejo de Estados de Carga

```kotlin
@Composable
fun UserListWithStates(
    users: LazyPagingItems<User>
) {
    LazyColumn {
        when (users.loadState.refresh) {
            is LoadState.Loading -> {
                item {
                    Box(
                        modifier = Modifier.fillMaxSize(),
                        contentAlignment = Alignment.Center
                    ) {
                        CircularProgressIndicator()
                    }
                }
            }
            is LoadState.Error -> {
                val error = users.loadState.refresh as LoadState.Error
                item {
                    ErrorView(
                        message = error.error.localizedMessage,
                        onRetry = { users.retry() }
                    )
                }
            }
            else -> {
                items(users.itemCount) { index ->
                    users[index]?.let { user ->
                        UserItem(user)
                    }
                }
                
                when (users.loadState.append) {
                    is LoadState.Loading -> {
                        item {
                            CircularProgressIndicator(
                                modifier = Modifier
                                    .fillMaxWidth()
                                    .padding(16.dp)
                            )
                        }
                    }
                    is LoadState.Error -> {
                        item {
                            RetryButton(onClick = { users.retry() })
                        }
                    }
                    else -> {}
                }
            }
        }
    }
}
```

## Errores Comunes

❌ **No usar transactions**
```kotlin
// MAL - operaciones no atómicas
if (loadType == LoadType.REFRESH) {
    userDao.clearAll()
}
userDao.insertAll(users)
```

✅ **Usar withTransaction**
```kotlin
database.withTransaction {
    if (loadType == LoadType.REFRESH) {
        userDao.clearAll()
    }
    userDao.insertAll(users)
}
```

❌ **No manejar endOfPaginationReached**
```kotlin
// MAL - siempre retorna false
MediatorResult.Success(endOfPaginationReached = false)
```

✅ **Verificar si hay más datos**
```kotlin
MediatorResult.Success(
    endOfPaginationReached = response.isEmpty()
)
```

## Recursos Adicionales

- [Paging 3 Codelab](https://developer.android.com/codelabs/android-paging)
- [Now in Android - Paging Example](https://github.com/android/nowinandroid)
- [Paging 3 Sample](https://github.com/android/architecture-components-samples/tree/main/PagingSample)

## Tags
#paging #remotemediator #offline-first #database #network

---
*Última actualización: 2025-01-14*
