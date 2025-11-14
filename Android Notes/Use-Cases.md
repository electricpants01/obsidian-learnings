# Use Cases / Interactors

## Descripción

Los Use Cases (también llamados Interactors) son componentes de la capa de dominio que encapsulan la lógica de negocio de la aplicación. Cada Use Case representa una acción específica que el usuario puede realizar.

### Componentes Principales

1. **Use Case Class**: Clase que encapsula una operación de negocio
2. **Input Parameters**: Parámetros necesarios para ejecutar la operación
3. **Output/Result**: Resultado de la operación

### Ventajas

- Lógica de negocio aislada y reutilizable
- Facilita el testing unitario
- Código más legible y mantenible
- Seguimiento del principio Single Responsibility
- Desacopla ViewModel de Repository

## Ejemplo de Implementación

```kotlin
// Use Case Simple
class GetUserByIdUseCase(
    private val userRepository: UserRepository
) {
    suspend operator fun invoke(userId: String): Result<User> {
        return userRepository.getUserById(userId)
    }
}

// Use Case con Lógica de Negocio
class GetActiveUsersUseCase(
    private val userRepository: UserRepository
) {
    suspend operator fun invoke(): Result<List<User>> {
        return userRepository.getUsers()
            .map { users ->
                users.filter { user ->
                    user.isActive && user.lastLogin.isRecent()
                }
            }
    }
}

// Use Case con Múltiples Repositorios
class GetUserProfileWithPostsUseCase(
    private val userRepository: UserRepository,
    private val postsRepository: PostsRepository
) {
    suspend operator fun invoke(userId: String): Result<UserProfile> {
        return try {
            val user = userRepository.getUserById(userId).getOrThrow()
            val posts = postsRepository.getPostsByUser(userId).getOrThrow()
            
            val profile = UserProfile(
                user = user,
                posts = posts,
                totalPosts = posts.size
            )
            
            Result.success(profile)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

// Use Case con Parámetros Complejos
class SearchUsersUseCase(
    private val userRepository: UserRepository
) {
    data class Params(
        val query: String,
        val filters: UserFilters = UserFilters(),
        val sortBy: SortOption = SortOption.NAME
    )
    
    suspend operator fun invoke(params: Params): Result<List<User>> {
        return userRepository.searchUsers(params.query)
            .map { users ->
                users
                    .filter { it.matchesFilters(params.filters) }
                    .sortedBy(params.sortBy)
            }
    }
}

// Uso en ViewModel
class UserViewModel(
    private val getUserByIdUseCase: GetUserByIdUseCase,
    private val getActiveUsersUseCase: GetActiveUsersUseCase
) : ViewModel() {
    
    private val _user = MutableStateFlow<User?>(null)
    val user: StateFlow<User?> = _user.asStateFlow()
    
    fun loadUser(userId: String) {
        viewModelScope.launch {
            getUserByIdUseCase(userId)
                .onSuccess { user -> _user.value = user }
                .onFailure { /* Handle error */ }
        }
    }
    
    fun loadActiveUsers() {
        viewModelScope.launch {
            getActiveUsersUseCase()
                .onSuccess { users -> /* Update state */ }
                .onFailure { /* Handle error */ }
        }
    }
}
```

## Use Cases con Flow

```kotlin
// Use Case que retorna Flow
class ObserveUserUseCase(
    private val userRepository: UserRepository
) {
    operator fun invoke(userId: String): Flow<User> {
        return userRepository.observeUser(userId)
    }
}

// Use Case que combina múltiples Flows
class ObserveUserDashboardUseCase(
    private val userRepository: UserRepository,
    private val statsRepository: StatsRepository
) {
    operator fun invoke(userId: String): Flow<UserDashboard> {
        return combine(
            userRepository.observeUser(userId),
            statsRepository.observeUserStats(userId)
        ) { user, stats ->
            UserDashboard(user, stats)
        }
    }
}

// Uso con Flow
class UserViewModel(
    private val observeUserUseCase: ObserveUserUseCase
) : ViewModel() {
    
    fun observeUser(userId: String): StateFlow<User?> {
        return observeUserUseCase(userId)
            .stateIn(
                scope = viewModelScope,
                started = SharingStarted.WhileSubscribed(5000),
                initialValue = null
            )
    }
}
```

## Documentación Oficial

- [Domain Layer](https://developer.android.com/topic/architecture/domain-layer)
- [Use Cases Best Practices](https://developer.android.com/topic/architecture/domain-layer#use-cases)
- [Architecture Components](https://developer.android.com/topic/architecture)

## Videos Recomendados

- [Use Cases in Clean Architecture](https://www.youtube.com/watch?v=B1xSb2eDDiA) - Philipp Lackner
- [Domain Layer Explained](https://www.youtube.com/watch?v=TPpOfSVSPZM) - Coding with Mitch
- [Clean Architecture Use Cases](https://www.youtube.com/watch?v=Nsjsiz2A9mg) - Android Developers

## Mejores Prácticas

1. **Un Use Case = Una Responsabilidad**: Cada Use Case debe hacer una sola cosa
2. **Nombrar por Acción**: Usar verbos como `Get`, `Create`, `Update`, `Delete`, `Observe`
3. **No depender de Android**: Use Cases deben ser Kotlin puro sin dependencias de Android
4. **Usar operator fun invoke()**: Permite llamar al Use Case como función
5. **Return Result o Flow**: Para manejo de errores consistente
6. **Testing fácil**: Deben ser fáciles de testear con repositorios falsos

## Patrones de Naming

```kotlin
// ✅ Nombres descriptivos con verbo
GetUserByIdUseCase
CreateNewUserUseCase
UpdateUserProfileUseCase
DeleteUserAccountUseCase
ObserveActiveUsersUseCase
SearchUsersByNameUseCase

// ❌ Nombres genéricos o ambiguos
UserUseCase
HandleUserUseCase
ProcessUserUseCase
```

## Errores Comunes

❌ **Lógica de UI en Use Case**
```kotlin
class GetUsersUseCase {
    suspend operator fun invoke(): List<UserUiModel> {
        // NO incluir lógica de UI en Use Case
        return repository.getUsers().map { it.toUiModel() }
    }
}
```

✅ **Mantener Use Case en Domain Layer**
```kotlin
class GetUsersUseCase {
    suspend operator fun invoke(): Result<List<User>> {
        // Retornar modelos de dominio
        return repository.getUsers()
    }
}
```

❌ **Use Case demasiado complejo**
```kotlin
class HandleUserOperationsUseCase {
    // Múltiples responsabilidades - MAL
    suspend fun getUser() { }
    suspend fun updateUser() { }
    suspend fun deleteUser() { }
}
```

✅ **Separar en Use Cases específicos**
```kotlin
class GetUserUseCase { }
class UpdateUserUseCase { }
class DeleteUserUseCase { }
```

❌ **Depender de Context o Android**
```kotlin
class GetUsersUseCase(
    private val context: Context // MAL
) {
    suspend operator fun invoke() {
        val sharedPrefs = context.getSharedPreferences(...)
    }
}
```

✅ **Mantener independencia de Android**
```kotlin
class GetUsersUseCase(
    private val userRepository: UserRepository, // Solo dependencias de dominio
    private val preferencesRepository: PreferencesRepository
) {
    suspend operator fun invoke(): Result<List<User>> {
        return userRepository.getUsers()
    }
}
```

## Testing de Use Cases

```kotlin
class GetUserByIdUseCaseTest {
    
    private lateinit var useCase: GetUserByIdUseCase
    private lateinit var repository: FakeUserRepository
    
    @Before
    fun setup() {
        repository = FakeUserRepository()
        useCase = GetUserByIdUseCase(repository)
    }
    
    @Test
    fun `invoke with valid id returns user`() = runTest {
        // Given
        val expectedUser = User(id = "1", name = "John")
        repository.addUser(expectedUser)
        
        // When
        val result = useCase("1")
        
        // Then
        assertTrue(result.isSuccess)
        assertEquals(expectedUser, result.getOrNull())
    }
    
    @Test
    fun `invoke with invalid id returns failure`() = runTest {
        // When
        val result = useCase("invalid")
        
        // Then
        assertTrue(result.isFailure)
    }
}
```

## Recursos Adicionales

- [Clean Architecture Guide](https://github.com/android/architecture-samples)
- [Domain Layer Documentation](https://developer.android.com/topic/architecture/domain-layer)
- [Use Cases Pattern](https://proandroiddev.com/why-you-need-use-cases-interactors-142e8a6fe576)

## Tags
#arquitectura #usecase #domain #cleanarchitecture #interactor

---
*Última actualización: 2025-01-14*
