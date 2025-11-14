# Clean Architecture

## Descripción

Clean Architecture es un patrón que separa la aplicación en capas independientes. Cada capa tiene responsabilidades específicas y depende solo de capas internas.

### Puntos Clave

- Domain Layer: Lógica de negocio y casos de uso
- Data Layer: Repositorios y fuentes de datos
- Presentation Layer: ViewModels y UI

## Ejemplo de Código

```kotlin
// Domain Layer
interface UserRepository {
    suspend fun getUsers(): List<User>
}

class GetUsersUseCase(private val repository: UserRepository) {
    suspend operator fun invoke() = repository.getUsers()
}
```

## Documentación Oficial

- [App Architecture Guide](https://developer.android.com/topic/architecture)
- [Recommended App Architecture](https://developer.android.com/topic/architecture/recommendations)

## Videos Recomendados

- [Clean Architecture](https://www.youtube.com/watch?v=Nsjsiz2A9mg)
- [Android Clean Architecture](https://www.youtube.com/watch?v=iJmJYy6pz9g)

## Mejores Prácticas

1. **Separación clara de capas**
2. **Dependencias hacia adentro**
3. **Interfaces para abstracciones**
4. **Testing independiente por capa**

## Tags
#arquitectura #clean #layers #separation

---
*Última actualización: 2025-01-14*
