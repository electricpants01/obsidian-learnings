# MVI (Model-View-Intent)

## Descripción

MVI es un patrón arquitectónico unidireccional que se basa en la arquitectura reactiva. Proporciona un flujo de datos predecible y facilita el manejo del estado de la aplicación.

### Puntos Clave

- Model: Estado inmutable de la UI
- View: Renderiza el estado y emite intenciones
- Intent: Representa las intenciones del usuario

## Ejemplo de Código

```kotlin
sealed class UserIntent {
    object LoadUsers : UserIntent()
    data class SearchUser(val query: String) : UserIntent()
}

data class UserState(
    val users: List<User> = emptyList(),
    val isLoading: Boolean = false
)
```

## Documentación Oficial

- [MVI Architecture](https://developer.android.com/topic/architecture/ui-layer)
- [Unidirectional Data Flow](https://developer.android.com/topic/architecture/ui-layer#udf)

## Videos Recomendados

- [MVI Architecture](https://www.youtube.com/watch?v=64rQ9GKphTg)
- [MVI with Kotlin Flow](https://www.youtube.com/watch?v=_J6LNX7j8Fo)

## Mejores Prácticas

1. **Estado inmutable siempre**
2. **Single source of truth con StateFlow**
3. **Intenciones claras y descriptivas**
4. **Reducers sin efectos secundarios**

## Tags
#arquitectura #mvi #unidirectional #statemanagement

---
*Última actualización: 2025-01-14*
