# MVVM (Model-View-ViewModel)

## Descripción

MVVM es un patrón arquitectónico que separa la lógica de presentación de la interfaz de usuario. Es el patrón recomendado por Google para el desarrollo de aplicaciones Android modernas.

### Puntos Clave

- Model: Representa los datos y la lógica de negocio
- View: La interfaz de usuario (Activities, Fragments, Composables)
- ViewModel: Intermediario que expone datos a la Vista de manera observable

## Ejemplo de Código

```kotlin
class UserViewModel : ViewModel() {
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users: StateFlow<List<User>> = _users.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _users.value = repository.getUsers()
        }
    }
}
```

## Documentación Oficial

- [Guide to app architecture](https://developer.android.com/topic/architecture)
- [ViewModel Overview](https://developer.android.com/topic/libraries/architecture/viewmodel)

## Videos Recomendados

- [Android Architecture Components](https://www.youtube.com/watch?v=5qlIPTDE274)
- [MVVM Pattern](https://www.youtube.com/watch?v=0EdF3R_PTDw)

## Mejores Prácticas

1. **Un ViewModel por pantalla**
2. **Exponer estado inmutable con StateFlow**
3. **No pasar Context al ViewModel**
4. **Usar ViewModelScope para coroutines**

## Tags
#arquitectura #mvvm #viewmodel #android

---
*Última actualización: 2025-01-14*
