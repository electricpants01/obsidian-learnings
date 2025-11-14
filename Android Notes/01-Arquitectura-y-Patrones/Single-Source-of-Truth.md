# Single Source of Truth (SSOT)

## Descripción

Single Source of Truth es un principio arquitectónico que establece que cada pieza de dato en la aplicación debe tener una única fuente autoritativa. Todos los demás accesos a ese dato deben referirse a esta fuente.

### Concepto Principal

**Una fuente = Un dato**

Cada tipo de dato debe tener exactamente una fuente de verdad en la aplicación, eliminando inconsistencias y simplificando el flujo de datos.

### Ventajas

- Elimina inconsistencias de datos
- Simplifica debugging y razonamiento sobre el código
- Facilita el testing
- Previene bugs de sincronización
- Clara jerarquía de datos

## Ejemplo de Implementación

```kotlin
// ❌ INCORRECTO: Múltiples fuentes de verdad
class UserViewModel {
    private val users = mutableListOf<User>()  // Lista local
    private val userCache = HashMap<String, User>()  // Cache separado
    // ¿Cuál es la fuente de verdad?
}

// ✅ CORRECTO: Una sola fuente de verdad
class UserViewModel(
    private val userRepository: UserRepository
) : ViewModel() {
    // Repository es la ÚNICA fuente de verdad
    val users: StateFlow<List<User>> = userRepository
        .observeUsers()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )
}

// Repository como SSOT
class UserRepositoryImpl(
    private val database: UserDatabase,
    private val api: UserApi
) : UserRepository {
    
    override fun observeUsers(): Flow<List<User>> {
        // La base de datos es la fuente de verdad
        return database.userDao().observeAll()
            .map { entities -> entities.map { it.toDomain() } }
    }
    
    override suspend fun refreshUsers() {
        try {
            // Actualiza la fuente de verdad
            val remoteUsers = api.getUsers()
            database.userDao().insertAll(remoteUsers.map { it.toEntity() })
            // No necesitamos retornar datos - los observadores se actualizan automáticamente
        } catch (e: Exception) {
            // Manejar error
        }
    }
}
```

## SSOT con Room y Network

```kotlin
// Base de datos como SSOT
class ArticleRepository(
    private val database: ArticleDatabase,
    private val api: NewsApi
) {
    // La UI siempre observa la base de datos (SSOT)
    fun observeArticles(): Flow<List<Article>> {
        return database.articleDao()
            .observeAll()
            .map { it.map { entity -> entity.toDomain() } }
    }
    
    // Refresh actualiza la SSOT
    suspend fun refresh() = withContext(Dispatchers.IO) {
        try {
            val articles = api.fetchArticles()
            // Actualizar la fuente de verdad
            database.articleDao().insertAll(
                articles.map { it.toEntity() }
            )
        } catch (e: Exception) {
            // Error handling
        }
    }
    
    // Búsqueda también usa la SSOT
    fun searchArticles(query: String): Flow<List<Article>> {
        return database.articleDao()
            .searchByTitle(query)
            .map { it.map { entity -> entity.toDomain() } }
    }
}
```

## SSOT con Paging 3

```kotlin
@OptIn(ExperimentalPagingApi::class)
class UserRepository(
    private val database: AppDatabase,
    private val api: UserApi
) {
    fun getUsers(): Flow<PagingData<User>> {
        return Pager(
            config = PagingConfig(pageSize = 20),
            remoteMediator = UserRemoteMediator(database, api),
            // La base de datos es la SSOT
            pagingSourceFactory = {
                database.userDao().pagingSource()
            }
        ).flow.map { pagingData ->
            pagingData.map { it.toDomain() }
        }
    }
}

// RemoteMediator sincroniza red -> DB (SSOT)
class UserRemoteMediator(
    private val database: AppDatabase,
    private val api: UserApi
) : RemoteMediator<Int, UserEntity>() {
    
    override suspend fun load(
        loadType: LoadType,
        state: PagingState<Int, UserEntity>
    ): MediatorResult {
        return try {
            val page = when (loadType) {
                LoadType.REFRESH -> 0
                LoadType.PREPEND -> return MediatorResult.Success(true)
                LoadType.APPEND -> {
                    val lastItem = state.lastItemOrNull()
                        ?: return MediatorResult.Success(true)
                    calculateNextPage(lastItem)
                }
            }
            
            val users = api.getUsers(page)
            
            database.withTransaction {
                if (loadType == LoadType.REFRESH) {
                    database.userDao().clearAll()
                }
                // Actualizar la fuente de verdad
                database.userDao().insertAll(
                    users.map { it.toEntity() }
                )
            }
            
            MediatorResult.Success(users.isEmpty())
        } catch (e: Exception) {
            MediatorResult.Error(e)
        }
    }
}
```

## Documentación Oficial

- [App Architecture Guide](https://developer.android.com/topic/architecture)
- [Data Layer](https://developer.android.com/topic/architecture/data-layer)
- [Offline-first Architecture](https://developer.android.com/topic/architecture/data-layer#offline-first)
- [Paging 3 with Network and Database](https://developer.android.com/topic/libraries/architecture/paging/v3-network-db)

## Videos Recomendados

- [Single Source of Truth](https://www.youtube.com/watch?v=pErTyQpNPr8) - Android Developers
- [Offline-First Architecture](https://www.youtube.com/watch?v=70WqJxei8qA) - Philipp Lackner
- [Repository Pattern SSOT](https://www.youtube.com/watch?v=W3IkBGj34UI) - Coding with Mitch

## Mejores Prácticas

1. **Base de datos como SSOT**: Para apps offline-first, la DB local es la fuente de verdad
2. **Repository coordina**: El Repository sincroniza entre fuentes pero la DB es autoritativa
3. **UI observa SSOT**: La UI siempre debe observar la fuente de verdad, nunca cachés temporales
4. **Actualizaciones unidireccionales**: Red → DB → UI (no directamente Red → UI)
5. **Transacciones atómicas**: Usa transactions para mantener consistencia

## Patrones Anti-SSOT

❌ **Múltiples cachés independientes**
```kotlin
class UserViewModel {
    private val localUsers = mutableListOf<User>()
    private val cachedUsers = HashMap<String, User>()
    // ¿Cuál es correcto si difieren?
}
```

✅ **Una sola fuente**
```kotlin
class UserViewModel(private val repository: UserRepository) {
    val users = repository.observeUsers()
        .stateIn(viewModelScope, SharingStarted.Lazily, emptyList())
}
```

❌ **Bypass de la fuente de verdad**
```kotlin
// MAL: Muestra datos de red directamente
viewModelScope.launch {
    val users = api.getUsers() // Bypassing SSOT
    _users.value = users
}
```

✅ **Actualizar SSOT**
```kotlin
// BIEN: Actualiza la fuente de verdad
viewModelScope.launch {
    repository.refresh() // Actualiza DB (SSOT)
    // La UI se actualiza automáticamente vía Flow
}
```

## SSOT con Multiple Entities

```kotlin
class DashboardRepository(
    private val database: AppDatabase,
    private val api: DashboardApi
) {
    // Combina múltiples fuentes de verdad
    fun observeDashboard(userId: String): Flow<Dashboard> {
        return combine(
            database.userDao().observeUser(userId),
            database.statsDao().observeStats(userId),
            database.postsDao().observeUserPosts(userId)
        ) { user, stats, posts ->
            Dashboard(
                user = user.toDomain(),
                stats = stats.toDomain(),
                recentPosts = posts.map { it.toDomain() }
            )
        }
    }
    
    suspend fun refreshDashboard(userId: String) {
        val dashboardData = api.getDashboard(userId)
        
        database.withTransaction {
            // Actualiza todas las fuentes de verdad atómicamente
            database.userDao().insert(dashboardData.user.toEntity())
            database.statsDao().insert(dashboardData.stats.toEntity())
            database.postsDao().insertAll(
                dashboardData.posts.map { it.toEntity() }
            )
        }
    }
}
```

## Testing con SSOT

```kotlin
class UserRepositoryTest {
    
    private lateinit var database: AppDatabase
    private lateinit var repository: UserRepository
    
    @Before
    fun setup() {
        database = Room.inMemoryDatabaseBuilder(
            context, AppDatabase::class.java
        ).build()
        repository = UserRepositoryImpl(database, fakeApi)
    }
    
    @Test
    fun `refresh updates database and observers receive new data`() = runTest {
        // Given
        val users = repository.observeUsers().test()
        
        // When
        repository.refresh()
        
        // Then
        val emittedUsers = users.awaitItem()
        assertEquals(expectedUsers, emittedUsers)
        // Verificar que la base de datos (SSOT) fue actualizada
        val dbUsers = database.userDao().getAll()
        assertEquals(expectedUsers.size, dbUsers.size)
    }
}
```

## Recursos Adicionales

- [Now in Android - SSOT Example](https://github.com/android/nowinandroid)
- [Architecture Samples](https://github.com/android/architecture-samples)
- [Offline-First Best Practices](https://developer.android.com/topic/architecture/data-layer#offline-first)

## Tags
#arquitectura #ssot #datalayer #offline-first #repository

---
*Última actualización: 2025-01-14*
