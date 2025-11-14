# Arquitectura de Jetcaster

## 📋 Índice

1. [Visión General](#visión-general)
2. [Patrón Arquitectónico](#patrón-arquitectónico)
3. [Capas de la Arquitectura](#capas-de-la-arquitectura)
4. [Estructura de Módulos](#estructura-de-módulos)
5. [Flujo de Datos](#flujo-de-datos)
6. [Componentes Clave](#componentes-clave)
7. [Tecnologías y Bibliotecas](#tecnologías-y-bibliotecas)
8. [Principios de Diseño](#principios-de-diseño)
9. [Gestión de Estado](#gestión-de-estado)
10. [Casos de Uso](#casos-de-uso)

---

## Visión General

Jetcaster es una aplicación de ejemplo que demuestra las mejores prácticas de arquitectura moderna para Android, construida completamente con **Jetpack Compose**. La aplicación implementa un patrón arquitectónico **Redux/MVI (Model-View-Intent)** con flujo de datos unidireccional (UDF - Unidirectional Data Flow).

### Características Arquitectónicas Principales

- ✅ **Arquitectura multicapa**: UI, Domain, Data
- ✅ **Flujo de datos unidireccional (UDF)**
- ✅ **Estados inmutables**
- ✅ **Programación reactiva con Kotlin Flows**
- ✅ **Inyección de dependencias con Dagger Hilt**
- ✅ **Modularización por características y capas**
- ✅ **Soporte multi-plataforma** (Mobile, TV, Wear OS)

---

## Patrón Arquitectónico

### Redux/MVI (Model-View-Intent)

La aplicación sigue un patrón Redux adaptado para Android, donde:

```
┌─────────────┐
│     UI      │ ──(Actions)──> ┌──────────────┐
│  (Compose)  │                │  ViewModel   │
└─────────────┘ <──(State)──── └──────────────┘
                                      │
                                      ▼
                                ┌──────────────┐
                                │   Use Cases  │
                                └──────────────┘
                                      │
                                      ▼
                                ┌──────────────┐
                                │ Repositories │
                                └──────────────┘
                                      │
                     ┌────────────────┴────────────────┐
                     ▼                                 ▼
              ┌─────────────┐                   ┌─────────────┐
              │  Database   │                   │   Network   │
              │   (Room)    │                   │  (OkHttp)   │
              └─────────────┘                   └─────────────┘
```

### Características del Patrón

1. **Estado Único**: Cada pantalla tiene un único `StateFlow<ViewState>` que contiene todo el estado de la UI
2. **Inmutabilidad**: Los estados son `@Immutable data class`, garantizando que no puedan modificarse
3. **Acciones**: Las interacciones del usuario se modelan como acciones selladas (`sealed interface`)
4. **Reactivo**: Uso extensivo de Kotlin Flows para actualizaciones de estado reactivas

---

## Capas de la Arquitectura

### 1. UI Layer (Capa de Presentación)

**Responsabilidades:**
- Renderizar la interfaz de usuario con Jetpack Compose
- Observar el estado del ViewModel
- Enviar acciones del usuario al ViewModel
- Manejar la navegación

**Componentes:**
- **Screens**: Funciones composables que representan pantallas completas
- **ViewModels**: Gestionan el estado y la lógica de presentación
- **UI States**: Data classes inmutables que representan el estado completo de una pantalla
- **Actions**: Eventos/intenciones del usuario modelados como sealed interfaces

**Ejemplo:**

```kotlin
@HiltViewModel
class HomeViewModel @Inject constructor(...) : ViewModel() {
    // Estado privado mutable
    private val _state = MutableStateFlow(HomeScreenUiState())
    
    // Estado público inmutable expuesto a la UI
    val state: StateFlow<HomeScreenUiState> get() = _state
    
    // Manejo de acciones del usuario
    fun onHomeAction(action: HomeAction) {
        when (action) {
            is HomeAction.CategorySelected -> onCategorySelected(action.category)
            is HomeAction.PodcastUnfollowed -> onPodcastUnfollowed(action.podcast)
            // ...
        }
    }
}

@Immutable
data class HomeScreenUiState(
    val isLoading: Boolean = true,
    val errorMessage: String? = null,
    val featuredPodcasts: ImmutableList<PodcastInfo> = persistentListOf(),
    val selectedHomeCategory: HomeCategory = HomeCategory.Discover,
    // ...
)

@Immutable
sealed interface HomeAction {
    data class CategorySelected(val category: CategoryInfo) : HomeAction
    data class PodcastUnfollowed(val podcast: PodcastInfo) : HomeAction
    // ...
}
```

### 2. Domain Layer (Capa de Dominio)

**Responsabilidades:**
- Contener la lógica de negocio reutilizable
- Orquestar datos de múltiples repositorios
- Transformar datos entre formatos
- Aplicar reglas de negocio

**Componentes:**
- **Use Cases**: Encapsulan lógica de negocio específica
- **Domain Models**: Modelos de datos del dominio de negocio

**Ejemplos de Use Cases:**
- `FilterableCategoriesUseCase`: Gestiona categorías filtrables
- `PodcastCategoryFilterUseCase`: Filtra podcasts por categoría

**Ventajas:**
- ✅ Reutilización de lógica entre múltiples ViewModels
- ✅ Testabilidad mejorada
- ✅ Separación clara de responsabilidades
- ✅ Código más limpio y mantenible

### 3. Data Layer (Capa de Datos)

**Responsabilidades:**
- Gestionar todas las fuentes de datos (local y remota)
- Proporcionar una única fuente de verdad para los datos
- Manejar sincronización de datos
- Cachear datos para uso offline

**Componentes:**

#### Repositories
Coordinan las fuentes de datos y exponen APIs limpias

```kotlin
class PodcastsRepository @Inject constructor(
    private val podcastStore: PodcastStore,
    private val episodeStore: EpisodeStore,
    private val categoryStore: CategoryStore,
    private val podcastFetcher: PodcastFetcher
) {
    suspend fun updatePodcasts(force: Boolean) {
        // Lógica de actualización
    }
}
```

#### Stores
Manejan operaciones CRUD específicas con la base de datos

- **PodcastStore**: Operaciones de podcasts
  - `followPodcast()`, `unfollowPodcast()`
  - `togglePodcastFollowed()`
  - `followedPodcastsSortedByLastEpisode()`

- **EpisodeStore**: Operaciones de episodios
  - `episodesInPodcasts()`
  - `deleteEpisode()`

- **CategoryStore**: Operaciones de categorías

#### Database (Room)
- **JetcasterDatabase**: Base de datos Room principal
- **DAOs**: Interfaces para acceso a datos
- **Entities**: Modelos de base de datos
- **TypeConverters**: Conversión de tipos complejos (DateTime, etc.)

#### Network
- **PodcastFetcher**: Obtiene y parsea RSS feeds
- **OkHttp**: Cliente HTTP
- **Rome**: Parser de RSS

---

## Estructura de Módulos

El proyecto está modularizado para mejorar la escalabilidad y el tiempo de compilación:

```
jetcaster/
├── mobile/              # App móvil Android
├── tv/                  # App para Android TV
├── wear/                # App para Wear OS
├── glancewidget/        # Widget con Glance
└── core/                # Módulos compartidos
    ├── data/            # Capa de datos
    ├── data-testing/    # Utilidades de testing para data
    ├── domain/          # Lógica de negocio
    ├── domain-testing/  # Utilidades de testing para domain
    └── designsystem/    # Sistema de diseño compartido
```

### Ventajas de la Modularización

- ✅ **Compilación paralela**: Módulos se compilan en paralelo
- ✅ **Reutilización**: Código compartido entre mobile/tv/wear
- ✅ **Separación de concerns**: Cada módulo tiene responsabilidades claras
- ✅ **Testabilidad**: Módulos pueden testearse independientemente
- ✅ **Escalabilidad**: Fácil agregar nuevas características

---

## Flujo de Datos

### Flujo Unidireccional (UDF)

```
1. Usuario interactúa con la UI
         │
         ▼
2. UI envía Action al ViewModel
         │
         ▼
3. ViewModel procesa la Action
         │
         ▼
4. ViewModel llama Use Cases/Repository
         │
         ▼
5. Repository obtiene/actualiza datos
         │
         ▼
6. Datos fluyen a través de Flows
         │
         ▼
7. ViewModel actualiza el State
         │
         ▼
8. UI observa cambios de State y re-renderiza
```

### Ejemplo Práctico: Seguir un Podcast

```kotlin
// 1. Usuario hace clic en "Seguir"
Button(onClick = { onAction(HomeAction.TogglePodcastFollowed(podcast)) })

// 2. UI envía acción
fun onHomeAction(action: HomeAction) {
    when (action) {
        is HomeAction.TogglePodcastFollowed -> 
            onTogglePodcastFollowed(action.podcast)
    }
}

// 3. ViewModel procesa
private fun onTogglePodcastFollowed(podcast: PodcastInfo) {
    viewModelScope.launch {
        podcastStore.togglePodcastFollowed(podcast.uri)
    }
}

// 4. Store actualiza la base de datos
suspend fun togglePodcastFollowed(podcastUri: String) {
    // Operación en Room DB
}

// 5. Flow observa cambios en DB
val subscribedPodcasts = podcastStore.followedPodcastsSortedByLastEpisode(limit = 10)

// 6. ViewModel combina flows y actualiza estado
combine(subscribedPodcasts, ...) { podcasts, ... ->
    HomeScreenUiState(
        featuredPodcasts = podcasts.map { it.asExternalModel() }
    )
}.collect { _state.value = it }

// 7. UI re-renderiza automáticamente
val viewState by viewModel.state.collectAsStateWithLifecycle()
```

---

## Componentes Clave

### ViewModels

**Responsabilidades:**
- Sobreviven a cambios de configuración
- Gestionan el estado de la UI
- Ejecutan lógica de negocio en coroutines
- Exponen estados y funciones a la UI

**Características:**
- Anotados con `@HiltViewModel` para inyección de dependencias
- Extienden `ViewModel` del Architecture Components
- Usan `viewModelScope` para gestión automática del ciclo de vida de coroutines

### State Management

**Principios:**
- Estados inmutables con `@Immutable` annotation
- Collections inmutables con `kotlinx.collections.immutable`
- Single source of truth con `StateFlow`

```kotlin
@Immutable
data class HomeScreenUiState(
    val isLoading: Boolean = true,
    val featuredPodcasts: ImmutableList<PodcastInfo> = persistentListOf(),
    // ...
)
```

### Dependency Injection

**Dagger Hilt** proporciona:
- Inyección automática en ViewModels, Repositories, etc.
- Gestión del ciclo de vida
- Scopes automáticos
- Testing facilitado

```kotlin
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val podcastsRepository: PodcastsRepository,
    private val podcastStore: PodcastStore,
    // ...
) : ViewModel()
```

---

## Tecnologías y Bibliotecas

### UI
- **Jetpack Compose**: Framework declarativo de UI
- **Material 3**: Componentes Material Design
- **Navigation Compose**: Navegación declarativa
- **Coil**: Carga de imágenes asíncrona

### Architecture Components
- **ViewModel**: Gestión de estado con conciencia del ciclo de vida
- **Lifecycle**: Componentes conscientes del ciclo de vida
- **StateFlow**: Flujos de estado reactivos

### Persistence
- **Room**: Base de datos SQLite con ORM
- **DataStore**: Almacenamiento de preferencias

### Networking
- **OkHttp**: Cliente HTTP
- **Rome**: Parser de RSS feeds

### Reactive Programming
- **Kotlin Coroutines**: Programación asíncrona
- **Kotlin Flow**: Streams de datos reactivos

### Dependency Injection
- **Dagger Hilt**: Framework DI basado en Dagger

### Testing
- **JUnit**: Testing unitario
- **Módulos `-testing`**: Utilidades de testing reutilizables

### Multi-platform
- **Horologist**: Toolkit para Wear OS
- **Media Toolkit**: Componentes de UI para apps de media en Wear
- **Glance**: Widgets con Compose

---

## Principios de Diseño

### SOLID Principles

1. **Single Responsibility**: Cada clase tiene una única responsabilidad
   - ViewModels: Solo gestión de estado de UI
   - Repositories: Solo coordinación de datos
   - Stores: Solo operaciones CRUD específicas

2. **Open/Closed**: Extensible sin modificación
   - Sealed interfaces para Actions permiten agregar nuevas acciones
   - Use cases son fácilmente extensibles

3. **Liskov Substitution**: Interfaces bien definidas
   - Abstracciones claras entre capas

4. **Interface Segregation**: Interfaces específicas
   - Stores separados por entidad (PodcastStore, EpisodeStore)

5. **Dependency Inversion**: Depender de abstracciones
   - ViewModels dependen de interfaces, no implementaciones concretas

### Clean Architecture Principles

- ✅ **Separation of Concerns**: Capas claramente definidas
- ✅ **Dependency Rule**: Dependencias apuntan hacia dentro (Domain es independiente)
- ✅ **Testability**: Cada capa es testeable independientemente
- ✅ **Flexibility**: Fácil cambiar implementaciones (ej: cambiar DB)

### Reactive Programming

- Uso de **Flows** para streams de datos
- **StateFlow** para estado compartido
- Operators como `combine`, `flatMapLatest`, `map` para transformaciones
- **Backpressure** manejado automáticamente

---

## Gestión de Estado

### State Hoisting

Los estados se elevan al nivel más alto necesario:

```kotlin
// ViewModel posee el estado
private val _state = MutableStateFlow(HomeScreenUiState())
val state: StateFlow<HomeScreenUiState> get() = _state

// UI observa el estado
val viewState by viewModel.state.collectAsStateWithLifecycle()
```

### Inmutabilidad

Garantizada por:
- `@Immutable` annotation en data classes
- `ImmutableList` en lugar de `List` mutable
- `val` en lugar de `var` para propiedades
- Copy para actualizaciones: `state.copy(isLoading = false)`

### Lifecycle Awareness

```kotlin
// collectAsStateWithLifecycle() respeta el ciclo de vida
val viewState by viewModel.state.collectAsStateWithLifecycle()
```

Beneficios:
- No memory leaks
- Recolección pausada cuando la UI no es visible
- Reinicio automático cuando la UI vuelve a ser visible

---

## Casos de Uso

### Ejemplo 1: Obtener Podcasts Seguidos

```kotlin
class HomeViewModel @Inject constructor(
    private val podcastStore: PodcastStore,
    // ...
) : ViewModel() {
    
    // Flow que emite podcasts seguidos
    private val subscribedPodcasts = 
        podcastStore.followedPodcastsSortedByLastEpisode(limit = 10)
            .shareIn(viewModelScope, SharingStarted.WhileSubscribed())
    
    init {
        viewModelScope.launch {
            subscribedPodcasts.collect { podcasts ->
                _state.value = _state.value.copy(
                    featuredPodcasts = podcasts.map { it.asExternalModel() }
                )
            }
        }
    }
}
```

### Ejemplo 2: Combinar Múltiples Flows

```kotlin
init {
    viewModelScope.launch {
        combine(
            subscribedPodcasts,
            selectedHomeCategory,
            refreshing,
            filterableCategories,
            // ...
        ) { podcasts, category, isRefreshing, categories ->
            HomeScreenUiState(
                featuredPodcasts = podcasts.map { it.asExternalModel() },
                selectedHomeCategory = category,
                isLoading = isRefreshing,
                filterableCategoriesModel = categories,
            )
        }.collect { _state.value = it }
    }
}
```

### Ejemplo 3: Manejo de Errores

```kotlin
init {
    viewModelScope.launch {
        combine(/* ... */)
            .catch { throwable ->
                emit(
                    HomeScreenUiState(
                        isLoading = false,
                        errorMessage = throwable.message,
                    )
                )
            }
            .collect { _state.value = it }
    }
}
```

---

## Ventajas de Esta Arquitectura

### Para Desarrollo
- ✅ **Código predecible**: Flujo unidireccional facilita el seguimiento
- ✅ **Debugging simplificado**: Estado centralizado
- ✅ **Reusabilidad**: Lógica compartida en Use Cases
- ✅ **Testabilidad**: Cada capa testeable independientemente
- ✅ **Escalabilidad**: Fácil agregar nuevas features

### Para Mantenimiento
- ✅ **Separación clara**: Cada capa tiene responsabilidades definidas
- ✅ **Bajo acoplamiento**: Cambios en una capa no afectan otras
- ✅ **Alta cohesión**: Código relacionado agrupado
- ✅ **Documentación por diseño**: La arquitectura documenta el propósito

### Para Performance
- ✅ **Lazy loading**: Flows solo activos cuando se observan
- ✅ **Caching**: Room provee caching automático
- ✅ **Lifecycle aware**: Recursos liberados cuando no se necesitan
- ✅ **Modularización**: Compilación paralela más rápida

---

## Recursos Adicionales

### Documentación Oficial
- [Guide to app architecture](https://developer.android.com/topic/architecture)
- [Jetpack Compose](https://developer.android.com/jetpack/compose)
- [Kotlin Flows](https://kotlinlang.org/docs/flow.html)
- [Dagger Hilt](https://dagger.dev/hilt/)

### Patrones y Prácticas
- [MVI Pattern](https://hannesdorfmann.com/android/model-view-intent/)
- [Unidirectional Data Flow](https://developer.android.com/topic/architecture#unidirectional-data-flow)
- [State Management in Compose](https://developer.android.com/jetpack/compose/state)

---

## Conclusión

La arquitectura de Jetcaster demuestra las mejores prácticas modernas para aplicaciones Android:

- **Redux/MVI** para flujo de datos predecible
- **Modularización** para escalabilidad
- **Reactive programming** para actualizaciones eficientes
- **Clean Architecture** para mantenibilidad
- **Multi-platform** para reutilización de código

Esta arquitectura es mantenible, testeable, escalable y sigue las recomendaciones oficiales de Google para desarrollo Android moderno.
