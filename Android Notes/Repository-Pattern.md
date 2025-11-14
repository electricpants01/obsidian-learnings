# Repository Pattern

## Descripción

El Repository Pattern es un patrón de diseño que abstrae el acceso a datos, proporcionando una interfaz limpia para que la capa de dominio acceda a los datos sin conocer su origen (red, base de datos, cache, etc.).

### Componentes Principales

1. **Repository Interface**: Define el contrato de acceso a datos
2. **Repository Implementation**: Implementa la lógica de acceso a datos
3. **Data Sources**: Fuentes de datos (local, remota)

### Ventajas

- Abstracción del origen de datos
- Facilita el testing con repositorios falsos
- Centraliza la lógica de acceso a datos
- Soporte para estrategias de caching
- Separación de responsabilidades clara

## Ejemplo de Implementación

```kotlin
// Data Sources
interface UserLocalDataSource {
    suspend fun getUsers(): List<UserEntity>
    suspend fun saveUsers(users: List<UserEntity>)
}

interface UserRemoteDataSource {
    suspend fun fetchUsers(): List<UserDto>
}

// Repository Interface (Domain Layer)
interface UserRepository {
    suspend fun getUsers(forceRefresh: Boolean = false): Result<List<User>>
    suspend fun getUserById(id: String): Result<User>
}

// Repository Implementation (Data Layer)
class UserRepositoryImpl(
    private val localDataSource: UserLocalDataSource,
    private val remoteDataSource: UserRemoteDataSource,
    private val mapper: UserMapper
) : UserRepository {
    
    override suspend fun getUsers(forceRefresh: Boolean): Result<List<User>> {
        return try {
            if (forceRefresh) {
                // Fetch from network
                val remoteUsers = remoteDataSource.fetchUsers()
                val entities = remoteUsers.map { mapper.dtoToEntity(it) }
                localDataSource.saveUsers(entities)
            }
            
            // Return from local database
            val localUsers = localDataSource.getUsers()
            val domainUsers = localUsers.map { mapper.entityToDomain(it) }
            Result.success(domainUsers)
            
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    override suspend fun getUserById(id: String): Result<User> {
        return try {
            val userEntity = localDataSource.getUserById(id)
            val domainUser = mapper.entityToDomain(userEntity)
            Result.success(domainUser)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

// Usage in ViewModel
class UserViewModel(
    private val userRepository: UserRepository
) : ViewModel() {
    
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users: StateFlow<List<User>> = _users.asStateFlow()
    
    fun loadUsers(forceRefresh: Boolean = false) {
        viewModelScope.launch {
            userRepository.getUsers(forceRefresh)
                .onSuccess { users -> _users.value = users }
                .onFailure { /* Handle error */ }
        }
    }
}
```

## Estrategias de Cache

```kotlin
class UserRepositoryImpl(
    private val localDataSource: UserLocalDataSource,
    private val remoteDataSource: UserRemoteDataSource
) : UserRepository {
    
    // Estrategia: Network First, then Cache
    override suspend fun getUsers(): Result<List<User>> {
        return try {
            // Intenta obtener de red primero
            val remoteUsers = remoteDataSource.fetchUsers()
            localDataSource.saveUsers(remoteUsers)
            Result.success(remoteUsers.map { it.toDomain() })
        } catch (e: Exception) {
            // Si falla, usa cache
            try {
                val cachedUsers = localDataSource.getUsers()
                Result.success(cachedUsers.map { it.toDomain() })
            } catch (cacheError: Exception) {
                Result.failure(e)
            }
        }
    }
    
    // Estrategia: Cache First, then Network
    override suspend fun getUsersCacheFirst(): Result<List<User>> {
        return try {
            val cachedUsers = localDataSource.getUsers()
            if (cachedUsers.isNotEmpty()) {
                // Actualiza en background
                viewModelScope.launch {
                    try {
                        val remoteUsers = remoteDataSource.fetchUsers()
                        localDataSource.saveUsers(remoteUsers)
                    } catch (e: Exception) {
                        // Silent fail
                    }
                }
                Result.success(cachedUsers.map { it.toDomain() })
            } else {
                // Cache vacío, obtiene de red
                val remoteUsers = remoteDataSource.fetchUsers()
                localDataSource.saveUsers(remoteUsers)
                Result.success(remoteUsers.map { it.toDomain() })
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

## Documentación Oficial

- [Repository Pattern Guide](https://developer.android.com/topic/architecture/data-layer)
- [Data Layer Architecture](https://developer.android.com/topic/architecture/data-layer)
- [Offline-first Architecture](https://developer.android.com/topic/architecture/data-layer#offline-first)

## Videos Recomendados

- [Repository Pattern Explained](https://www.youtube.com/watch?v=W3IkBGj34UI) - Coding with Mitch
- [Data Layer in Android](https://www.youtube.com/watch?v=6LbPJiDPDL8) - Philipp Lackner
- [Clean Architecture Data Layer](https://www.youtube.com/watch?v=gZ0c9TBwUr4) - Android Developers

## Mejores Prácticas

1. **Interfaz en Domain Layer**: Define la interfaz en el dominio, implementación en data
2. **Result wrapper**: Usa Result o sealed classes para manejar éxito/error
3. **Mappers dedicados**: Separa entidades de red, DB y dominio
4. **Estrategia de cache clara**: Define cuándo usar cache vs network
5. **Inyección de dependencias**: Usa Hilt/Dagger para proveer repositorios

## Patrones de Caching

| Estrategia | Cuándo Usar | Ventajas | Desventajas |
|-----------|-------------|----------|-------------|
| Network First | Datos que cambian frecuentemente | Datos siempre actuales | Requiere conexión |
| Cache First | Datos estáticos o modo offline | Rápido, funciona offline | Puede estar desactualizado |
| Stale-While-Revalidate | Balance entre velocidad y actualidad | Rápido + eventual consistency | Complejidad media |

## Errores Comunes

❌ **No abstraer el origen de datos**
```kotlin
class UserViewModel {
    fun loadUsers() {
        // Acceso directo a data source - MAL
        val users = database.userDao().getUsers()
    }
}
```

✅ **Usar Repository**
```kotlin
class UserViewModel(private val repository: UserRepository) {
    fun loadUsers() {
        viewModelScope.launch {
            repository.getUsers()
                .onSuccess { users -> /* Update UI */ }
        }
    }
}
```

❌ **Lógica de negocio en Repository**
```kotlin
class UserRepositoryImpl : UserRepository {
    override suspend fun getAdultUsers(): List<User> {
        // Lógica de negocio en Repository - MAL
        return getUsers().filter { it.age >= 18 }
    }
}
```

✅ **Lógica en Use Case**
```kotlin
class GetAdultUsersUseCase(private val repository: UserRepository) {
    suspend operator fun invoke(): List<User> {
        return repository.getUsers().filter { it.age >= 18 }
    }
}
```

## Recursos Adicionales

- [Repository Pattern in Android](https://github.com/android/architecture-samples)
- [Data Layer Best Practices](https://developer.android.com/topic/architecture/data-layer#best-practices)
- [Offline-First Apps](https://developer.android.com/topic/architecture/data-layer/offline-first)

## Tags
#arquitectura #repository #datalayer #caching #android

---
*Última actualización: 2025-01-14*
