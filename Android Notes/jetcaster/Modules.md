###  Core Module

El Core module, tiene otros mas submodulos, como:

#### Data

![[Pasted image 20251022102034.png]]

tiene codigo relacionado con base de datos (dao, entities, JetcasterDatabase).
Tambien usan inyeccion de dependencias en un archivo `DataDiModule`.
El manejo de traer datos desde la base de datos es usando flows, lo que lo hace reactivo.

#### Data testing

Contiene clases, un wrapper a la interfaz `PodcastStore` que ayudaran a la hora de crear unit tests. 
Esto lo hacen on `CategoryStore`, `EpisodeStore` y `PodcastStore`.

#### Systemdesign

Este modulo esta hecho para tener componentes que se puedan usar en cualquier modulo de la aplicacion, como por ejemplo `PodcastImage`

#### Domain

This module contains different user actions (Use Cases), como ***FilterableCategoriesUseCase***, `GetLatestFollowedEpisodesUseCase` y `PodcastCategoryFilterUseCase`. 
Ademas de tener Models (modelos) con @Immutable que se usaran en el viewmodel y en los mismos @composable views.
Tambien tenemos modelos para el Player `PlayerEpisode`, el reproductor de audio, ademas tenemos una clase fake del EpisodePlayer, que se llama `MockEpisodePlayer`


#### Domain testing

Este modulo tiene PreviewData, es decir Dummy data, or hardcoded data that would facilitate rendering the UI previews.
