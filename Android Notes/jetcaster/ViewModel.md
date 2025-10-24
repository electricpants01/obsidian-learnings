
	Empezaremos con `HomeViewModel`, el viewmodel se relaciona con el repositorio y con varios Stores y el reproductor (Player).
`HomeViewModel` -> `podcastsRepository`

De la Manera en que se maneja los estados es a traves de `MutableStateFlow`, como `selectedLibraryPodcast`.

```
// Holds our view state which the UI collects via [state]  
private val _state = MutableStateFlow(HomeScreenUiState())

val state: StateFlow<HomeScreenUiState>  
    get() = _state
```

dentro del init, usan el `Combine`, el cual actualiza a los ultimos valores de las variables para generar un view state actualizado

```
viewModelScope.launch {  
    // Combines the latest value from each of the flows, allowing us to generate a  
    // view state instance which only contains the latest values.    com.example.jetcaster.core.util.combine(  
        homeCategories,  
        selectedHomeCategory,  
        subscribedPodcasts,  
        refreshing,  
        _selectedCategory.flatMapLatest { selectedCategory ->  
            filterableCategoriesUseCase(selectedCategory)  
        },  
        _selectedCategory.flatMapLatest {  
            podcastCategoryFilterUseCase(it)  
        },  
        subscribedPodcasts.flatMapLatest { podcasts ->  
            episodeStore.episodesInPodcasts(  
                podcastUris = podcasts.map { it.podcast.uri },  
                limit = 20,  
            )  
        },  
    ) {  
            homeCategories,  
            homeCategory,  
            podcasts,  
            refreshing,  
            filterableCategories,  
            podcastCategoryFilterResult,  
            libraryEpisodes,  
        ->  
  
        _selectedCategory.value = filterableCategories.selectedCategory  
  
        // Override selected home category to show 'DISCOVER' if there are no  
        // featured podcasts        selectedHomeCategory.value =  
            if (podcasts.isEmpty()) HomeCategory.Discover else homeCategory  
  
        HomeScreenUiState(  
            isLoading = refreshing,  
            homeCategories = homeCategories,  
            selectedHomeCategory = homeCategory,  
            featuredPodcasts = podcasts.map { it.asExternalModel() }.toPersistentList(),  
            filterableCategoriesModel = filterableCategories,  
            podcastCategoryFilterResult = podcastCategoryFilterResult,  
            library = libraryEpisodes.asLibrary(),  
        )  
    }.catch { throwable ->  
        emit(  
            HomeScreenUiState(  
                isLoading = false,  
                errorMessage = throwable.message,  
            ),  
        )  
    }.collect {  
        _state.value = it  
    }}
```


Luego tenemos las Acciones o Eventos, puedes llamarlas de ambas maneras. son acciones que el usuario realiza y deben realizarse dentro del viewmodel,

Screen Ui  -> Event -> Viewmodel ->

ViewModel -> SideEffect -> Screen Ui

A side effect es cuando el viewmodel manda instrucciones al screen ui, como por ejemplo despues de registrar un usuario, queremos movernos a otra pantalla (vista), otro ejemplo seria mostrar un snackbar para confirmar la accion del usuario.

Y los eventos es cuando el usuario rellena un form o presiona un boton, se toman esas acciones y las realizamos dentro del viewmodel.


