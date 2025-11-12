# Arquitectura del Módulo Digital Collections

## Resumen Ejecutivo

El módulo de Digital Collections utiliza una **arquitectura moderna de Android** basada en principios de clean architecture, flujo de datos unidireccional (UDF), y las mejores prácticas recomendadas por Google para el desarrollo de aplicaciones Android.

---

## 1. Arquitectura por Capas (Layered Architecture)

El módulo se organiza en tres capas principales que siguen el principio de separación de responsabilidades:

### 1.1 UI Layer (Capa de Presentación)

**Responsabilidades:**
- Mostrar la interfaz de usuario
- Escuchar y responder a interacciones del usuario
- Comunicarse con el usuario

**Componentes:**
- Activities y Fragments
- Composables de Jetpack Compose
- Objetos de Estado (ViewState)
- Navegación
- Animaciones
- Características de accesibilidad
- Clases del framework de Android

**Principios Clave:**
- **State Hoisting**: Los composables son lo más sin estado posible
- **Una Pantalla → Un ViewModel → Un UI State**: Cada pantalla tiene su propio ViewModel y clase de estado
- **Inmutabilidad del Estado**: Uso de `data classes` con propiedades `val` y anotación `@Immutable`
- **Separación UI/Business Logic**: La lógica UI permanece en la UI, la lógica de negocio en ViewModel/Domain/Data

### 1.2 Data Layer (Capa de Datos)

**Responsabilidades:**
- Crear, leer, actualizar y proporcionar datos
- Actuar como fuente única de verdad

**Componentes:**

#### Data Sources (Fuentes de Datos)
- Nivel más bajo de la arquitectura
- Cada data source trabaja con **una sola** fuente de datos
- Nunca accedidos directamente por otras capas
- Ejemplos: Operaciones Apollo GraphQL, APIs Retrofit, bases de datos Room, SharedPreferences

#### Repositories (Repositorios)
- Realizan lógica de negocio para transformar datos "crudos" en modelos de negocio
- Pueden depender de múltiples data sources
- Resuelven conflictos de datos potenciales
- Proporcionan una fuente única de verdad a las capas superiores

#### Business Models (Modelos de Negocio)
- Clases de datos que contienen solo la información necesaria para las necesidades de la feature
- Refinan datos crudos (modelos de red/base de datos) en modelos de negocio
- Diferentes de los modelos UI (transformación adicional en ViewModel)

```kotlin
// Ejemplo de transformación: Raw Apollo Model → Business Model → UI State Model

// 1. Función de transformación de modelo Apollo a modelo de negocio
fun CollectibleDetailsQuery.Data.toCollectibleItem(): CollectibleItem? {
    return this.collections.collectible?.let {
        CollectibleItem(
            id = it.id,
            title = it.title,
            price = currencyAmountOrNull(
                it.purchase?.price?.priceFragment?.amount,
                it.purchase?.price?.priceFragment?.currency.toString(),
            ),
            purchaseDate = it.purchase?.date?.let { millis ->
                EbayDateUtils.parseIso8601DateTime(millis.toString())
            },
            // ... más campos
        )
    }
}

// 2. Business Model
data class CollectibleItem(
    val id: String,
    val title: String,
    val price: CurrencyAmount? = null,
    val purchaseDate: Date? = null,
    // ... más campos
)

// 3. UI State Model (transformado en ViewModel)
@Immutable
data class CollectibleItemUiState(
    val id: String = "",
    val title: String = "",
    val price: String? = "$100.00", // Ya formateado para UI
    val aspects: List<Pair<UiText, UiText>> = emptyList(),
    // ... más campos
)
```

### 1.3 Domain Layer (Capa de Dominio)

**Responsabilidades:**
- Encapsular lógica de negocio compleja
- Proporcionar middleware entre UI y Data layers
- Facilitar reutilización de código

**Componentes:**
- **Use Cases**: Clases que encapsulan lógica de negocio específica
- Helpers, Utils, Interactors

**Beneficios:**
- Evita duplicación de código
- Mejora legibilidad en ViewModels
- Facilita testing (pueden ser mockeados/stubbed)
- Mejor separación de responsabilidades

**Convención de Nomenclatura:**
- `[Verbo en presente] + [Sustantivo] + UseCase`
- Ejemplo: `ToggleBookmarkUseCase`, `FetchCollectibleItemUseCase`

---

## 2. Patrón UDF (Unidirectional Data Flow)

El módulo implementa **flujo de datos unidireccional** usando una variante del patrón **Model-View-Intent (MVI)**.

### 2.1 ¿Qué es UDF?

El aspecto definitorio de UDF es que **los datos siempre fluyen en una dirección** a medida que se mueven a través de las capas de la arquitectura.

![Flujo UDF](https://developer.android.com/static/topic/libraries/architecture/images/mad-arch-ui-udf-in-action.png)

### 2.2 Componentes del Patrón UDF

#### ViewState (Estado de Vista)
```kotlin
@Immutable
data class CollectibleItemViewState(
    val uiState: UiState,
    val collectibleItemUiState: CollectibleItemUiState,
    val isBookmarked: Boolean = false,
    val currentMenu: CollectionsOptionsMenuState? = null
)
```

#### Events (Eventos)
```kotlin
sealed interface CollectibleItemDetailsEvent {
    data class UpdateCurrentImageIndex(val index: Int) : CollectibleItemDetailsEvent
    data class UpdateActionBar(val title: String?, val isUpAsClose: Boolean) : CollectibleItemDetailsEvent
    data class ToggleBookmark(val isBookmarked: Boolean) : CollectibleItemDetailsEvent
}
```

#### Effects/SideEffects (Efectos Secundarios)
```kotlin
sealed interface CollectibleItemDetailsEffect {
    data class ShowSnackBar(val message: UiText) : CollectibleItemDetailsEffect
}
```

### 2.3 Flujo de Datos UDF

1. **Usuario interactúa con UI** → UI envía **Event** al ViewModel
2. **ViewModel recibe evento** → Realiza trabajo necesario (puede interactuar con Data Layer)
3. **Data Layer responde** → Proporciona datos actualizados
4. **ViewModel actualiza estado** → Dos opciones:
   - Actualiza `ViewState` (preferido, recomposición automática)
   - Emite un `Effect` (para casos específicos como snackbars, navegación)
5. **UI reacciona** → Se recompone automáticamente o responde al efecto

### 2.4 Estado vs Efectos Secundarios

**Usar actualización de estado cuando:**
- Necesitas mostrar datos en la UI
- Los datos deben persistir durante la vida de la pantalla
- La UI debe reaccionar a cambios de datos

**Usar efectos secundarios solo para:**
- Mostrar snackbars o toasts
- Navegar a otra pantalla
- Abrir diálogos o bottom sheets
- Lanzar activities o fragments
- Acciones one-off que no deben persistir

### 2.5 CollectiblesScaffold

**CollectiblesScaffold** es un delegate de Kotlin que facilita la implementación del patrón UDF:

```kotlin
class CollectibleItemViewModel @Inject constructor(
    private val collectibleItemRepository: CollectibleItemRepository,
) : ViewModel(),
    CollectiblesScaffold<
        CollectibleItemViewState,           // ViewState type
        CollectibleItemDetailsEvent,        // Event type
        CollectibleItemDetailsEffect        // Effect type
    > by collectiblesScaffold(
        initialState = CollectibleItemViewState(
            uiState = UiState.INITIAL,
            collectibleItemUiState = CollectibleItemUiState(),
            currentMenu = null,
        )
    ) {

    override fun onEvent(event: CollectibleItemDetailsEvent) {
        when (event) {
            is CollectibleItemDetailsEvent.UpdateCurrentImageIndex -> 
                updateCurrentImageIndex(event.index)
            is CollectibleItemDetailsEvent.ToggleBookmark -> 
                toggleBookmark(event.isBookmarked)
        }
    }

    // Actualizar estado
    private fun updateBookmarkState(isBookmarked: Boolean) {
        updateViewState {
            it.copy(bookmarkingEnabled = isBookmarked)
        }
    }

    // Emitir efecto
    private fun showSnackBar(message: UiText) {
        viewModelScope.emitViewEffect(
            CollectibleItemDetailsEffect.ShowSnackBar(message)
        )
    }
}
```

---

## 3. Herramientas y Tecnologías

### 3.1 UI
- **Jetpack Compose**: Framework de UI declarativo y reactivo
- **State Hoisting**: Patrón para composables sin estado
- **Material Design Components**: Componentes UI consistentes

### 3.2 Arquitectura
- **ViewModel**: Gestión de estado y lógica de negocio
- **StateFlow**: Manejo de estado reactivo
- **CollectiblesScaffold**: Delegate para patrón UDF

### 3.3 Asincronía
- **Kotlin Coroutines**: Programación asíncrona
- **Flow**: Streams de datos reactivos
- **SharedFlow**: Para eventos one-off

### 3.4 Navegación
- **Navigation Component**: Navegación type-safe
- **Serializable Routes**: Rutas de navegación type-safe con `@Serializable`

### 3.5 Inyección de Dependencias
- **Dagger**: Framework de inyección de dependencias

### 3.6 Data
- **Apollo GraphQL**: Cliente GraphQL para APIs
- **Room**: Base de datos local
- **Retrofit**: Cliente HTTP (si aplica)

---

## 4. Patrones de Inicialización

### 4.1 Patrón InitContent con Refresh Trigger

**Refresh Function:**
```kotlin
private val refreshTrigger = MutableSharedFlow<Unit>()

private fun refresh() {
    viewModelScope.launch {
        refreshTrigger.emit(Unit)
    }
}
```

**Data Flow Observando Refresh Trigger:**
```kotlin
private val dataUpdates = refreshTrigger.flatMapLatest {
    dataRepository.fetchData()
}.onEach { data ->
    updateViewState { it.copy(data = data) }
}.launchIn(viewModelScope)
```

**Evento InitContent:**
```kotlin
override fun handleEvent(event: MyScreenEvent) {
    when (event) {
        is MyScreenEvent.InitContent -> initContent(event.input)
    }
}

private fun initContent(input: MyScreenInput) {
    _input.value = input
    refresh() // Trigger inicial de datos
}
```

### 4.2 Actualizaciones Reactivas

**Pattern de Filtros:**
```kotlin
private val filterUpdates = viewState.mapNotNull { it.filterState }
    .distinctUntilChanged()
    .drop(1) // Saltar emisión inicial
    .onEach { refresh() }
    .launchIn(viewModelScope)
```

**Flujos Reactivos Complejos:**
```kotlin
private val graderUpdates = viewState.mapNotNull { it.gradeFilterState?.selectedGraderId }
    .distinctUntilChanged()
    .flatMapLatest { graderId ->
        dataProvider.graderGrades(graderId)
    }
    .onEach { gradeData ->
        updateViewState { it.copy(gradeOptions = gradeData) }
    }
    .launchIn(viewModelScope)
```

---

## 5. Componentes Auto-Contenidos (Self-Contained Components)

Componentes reutilizables que gestionan su propio estado y siguen el mismo patrón UDF que las pantallas.

### 5.1 Factory Interface

```kotlin
interface ConditionSelectionFactory {
    @Composable
    fun HostedComponent(
        modifier: Modifier,
        viewModelStoreOwner: ViewModelStoreOwner,
        input: ConditionSelectionInput,
        showEntryPointField: Boolean,
        isSheetVisible: Boolean,
        onDismiss: () -> Unit,
        onSelectionChanged: (selection: ItemConditionModel) -> Unit,
    )
}

@Immutable
data class ConditionSelectionInput(
    val categoryId: String,
    val itemConditionModel: ItemConditionModel?,
)
```

### 5.2 Factory Implementation

Maneja setup, wiring, y lifecycle management del componente:

```kotlin
class ConditionSelectionFactoryImpl @Inject constructor(
    private val conditionSelectionViewModelFactory: ConditionSelectionViewModel.Factory,
) : ConditionSelectionFactory {

    @Composable
    override fun HostedComponent(
        modifier: Modifier,
        viewModelStoreOwner: ViewModelStoreOwner,
        input: ConditionSelectionInput,
        showEntryPointField: Boolean,
        isSheetVisible: Boolean,
        onDismiss: () -> Unit,
        onSelectionChanged: (selection: ItemConditionModel) -> Unit,
    ) {
        // ViewModel creation con scoping apropiado
        val viewModel = viewModel<ConditionSelectionViewModel>(
            viewModelStoreOwner = viewModelStoreOwner,
            factory = ConditionSelectionViewModel.createFactory(
                assistedFactory = conditionSelectionViewModelFactory,
            ),
        )
        
        val viewState by viewModel.viewState.collectAsStateWithLifecycle()
        
        // Manejo de side effects - navegación interna y comunicación externa
        LaunchedEffect(viewModel.sideEffects) {
            viewModel.sideEffects.collect { sideEffect ->
                when (sideEffect) {
                    is ConditionSelectionSideEffect.ConfirmSelection -> {
                        onSelectionChanged(sideEffect.itemCondition)
                        onDismiss()
                    }
                    // ... más efectos
                }
            }
        }

        // Inicialización usando patrón InitContent
        LaunchedEffect(input) {
            viewModel.handleEvent(ConditionSelectionEvent.InitContent(input))
        }

        // Renderizado de UI del componente
        if (showEntryPointField) {
            ConditionFieldContent(/* ... */)
        }
    }
}
```

### 5.3 Beneficios de Componentes Auto-Contenidos

- **Reutilizabilidad**: Pueden usarse en diferentes pantallas con comportamiento consistente
- **Aislamiento**: Estado del componente completamente separado del estado de la pantalla padre
- **Testabilidad**: Lógica del componente puede probarse independientemente
- **Contratos Claros**: Factory interface y callbacks definen límites de comunicación claros
- **Mantenibilidad**: Cambios internos no afectan pantallas consumidoras
- **Consistencia**: Sigue los mismos patrones arquitectónicos que las pantallas

---

## 6. Navegación

### 6.1 Type-Safe Navigation

Definición de rutas usando `@Serializable`:

```kotlin
@Serializable
data class AddNotesRoute(
    val collectibleId: String?,
    val note: String? = null,
)
```

### 6.2 Navegación Helper

```kotlin
fun NavController.navigateToAddNotesScreen(
    addNotesRoute: AddNotesRoute, 
    navOptions: NavOptions? = null
) = navigate(addNotesRoute, navOptions)
```

### 6.3 Screen Destination

```kotlin
fun NavGraphBuilder.addToNotesScreen(
    viewModelFactory: ViewModelProvider.Factory,
    navHostController: NavHostController,
    updateActionBar: (ActionBarState) -> Unit,
    showCollectiblesSnackBar: (CollectiblesSnackBarState) -> Unit,
) {
    composable<AddNotesRoute> { backStackEntry ->
        // Extracción de parámetros type-safe
        val initialData = backStackEntry.toRoute<AddNotesRoute>()

        // Creación de ViewModel con scoping apropiado
        val viewModel = viewModel<AddNotesViewModel>(
            viewModelStoreOwner = backStackEntry,
            factory = viewModelFactory,
            key = initialData.collectibleId
        )

        val viewState = viewModel.viewState.collectAsStateWithLifecycle().value

        // Inicialización usando patrón InitContent
        LaunchedEffect(Unit) {
            viewModel.handleEvent(AddNotesEvent.InitContent(initialData))
        }

        AddNotesScreen(
            state = viewState,
            handleEvent = viewModel::handleEvent,
            sideEffects = viewModel.sideEffects,
            onDismiss = { /* ... */ },
        )
    }
}
```

---

## 7. Testing

### 7.1 Mejores Prácticas Generales

- Preferir **fakes** sobre **mocks**
- Usar **MockK** sobre Mockito
- Usar **Turbine** para testear flows
- Preferir **unit tests UI con Robolectric** sobre instrumented tests
- Reducir código redundante con objetos de datos dummy reutilizables
- Usar **fixtures** cuando sea posible

### 7.2 Page Object Pattern

Patrón de diseño para testing UI:

**Definición de Page Object:**
```kotlin
class CollectibleLibraryPageObject(private val composeTestRule: ComposeTestRule) {

    fun folderMaxNotice(): SemanticsNodeInteraction = composeTestRule
        .onNodeWithText(
            "Folder limit reached. Delete a folder to create a new one.", 
            useUnmergedTree = true
        )

    fun folderListItemCount(count: Int): SemanticsNodeInteraction = composeTestRule
        .onNodeWithText("($count)", useUnmergedTree = true)

    fun generalErrorSnackBarMessage(): Matcher<View> = allOf(
        ViewMatchers.withId(com.google.android.material.R.id.snackbar_text),
        ViewMatchers.withText("Changes could not be saved, please try again.")
    )
}
```

**Uso en Tests:**
```kotlin
class CollectibleLibraryUiTest {
    private val collectibleLibraryPageObject = 
        CollectibleLibraryPageObject(composeTestRule)

    @Test
    fun generalErrorSnackBarMessageDisplayedOnCreateFolderError() {
        launchCollectionActivity()

        createRenameBottomSheetPageObject.run {
            textField().performClick().performTextReplacement("My folder")
            saveButton().performClick()
        }

        onView(collectibleLibraryPageObject.generalErrorSnackBarMessage())
            .checkDisplayed()
    }
}
```

---

## 8. Estructura de Módulos

El proyecto Digital Collections se divide en varios sub-módulos:

- **digitalCollectionsImpl**: Implementación principal del módulo
- **digitalCollectionsShared**: Código y recursos compartidos
- **digitalCollectionsApp**: Aplicación de demostración/sample
- **digitalCollectionsGradedCard**: Funcionalidad específica de tarjetas graduadas
- **digitalCollectionsIHS**: Integración con IHS (Item History Service)
- **digitalCollectionsTestSupport**: Utilidades y helpers para testing
- **digitalCollectionsInternalTestHelper**: Helpers internos de testing

---

## 9. Mejores Prácticas

### 9.1 Convenciones de Nomenclatura

**Events:**
- ✅ DO: Nombrar según la **acción solicitada**: `AddToFolder`, `ToggleBookmark`
- ⛔ DON'T: Nombrar según la interacción UI: `AddToFolderButtonClick`

**Effects:**
- ✅ DO: Nombrar según el **resultado esperado**: `ShowSnackBar(message: UiText)`
- ⛔ DON'T: Nombrar según el trigger: `OnItemAddedToFolder(message: UiText)`

**General:**
- ✅ DO: Nombres claros y concisos
- ⛔ DON'T: Usar "Event" o "Effect" en los nombres

### 9.2 State Management

- **Inmutabilidad**: Usar `data classes` con `val` properties
- **Single Source of Truth**: ViewModel como única fuente de verdad
- **No modificar estado directamente en UI**: Siempre a través de eventos
- **Usar `@Immutable`** solo cuando se cumplen las reglas de inmutabilidad

### 9.3 UDF Usage

- Aplicar UDF a nivel de pantalla
- Usar UDF consistentemente en toda la pantalla
- Evitar cadenas de eventos/efectos
- Preferir actualización de estado sobre efectos
- No usar eventos/efectos para comunicación puramente UI

### 9.4 Navigation Hygiene

- **No pasar ViewModels a composables Screen**
- **No pasar componentes UDF más allá del Screen composable**
- Aplicar **State Hoisting** en composables hijos
- Usar navegación type-safe con rutas `@Serializable`

---

## 10. Referencias

### Documentación Android
- [Android Developer's Guide to App Architecture](https://developer.android.com/topic/architecture)
- [State hoisting](https://developer.android.com/develop/ui/compose/state#state-hoisting)
- [UI Layer](https://developer.android.com/topic/architecture/ui-layer)
- [Data Layer](https://developer.android.com/topic/architecture/data-layer)
- [Business Models](https://developer.android.com/topic/architecture/data-layer#business-models)

### Artículos y Recursos
- [MVI Architecture with Kotlin Flows and Channels](https://proandroiddev.com/mvi-architecture-with-kotlin-flows-and-channels-d36820b2028d)
- [Are one-time events an anti-pattern?](https://youtu.be/njchj9d_Lf8)

### Documentación Interna
- `digitalCollections/docs/CollectiblesArchitectureAndBestPractices.md` - Documentación detallada completa

---

## Conclusión

La arquitectura del módulo Digital Collections representa una implementación moderna y robusta de las mejores prácticas de Android, combinando:

- **Clean Architecture** con separación clara de capas
- **UDF (Unidirectional Data Flow)** para manejo predecible del estado
- **Jetpack Compose** para UI declarativa y reactiva
- **Type-Safe Navigation** para navegación segura
- **Dependency Injection** con Dagger para código testeable
- **Componentes Reutilizables** con patrón Factory

Esta arquitectura está optimizada para:
- ✅ Escalabilidad
- ✅ Mantenibilidad
- ✅ Testabilidad
- ✅ Legibilidad
- ✅ Consistencia
- ✅ Productividad del equipo
