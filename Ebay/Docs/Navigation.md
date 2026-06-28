# Navigation

## CollectiblesNavHost

All screens are registered in `CollectiblesNavHost` in `view/navigation/CollectiblesNavHost.kt`.

| Screen | Route | Navigation File | ViewModel / Factory |
|---|---|---|---|
| **TabHost** (main hub) | `TabHostRoute` (data object) | `tabHostScreen()` | `TabHostViewModel` |
| **Collectible Item Details** | `CollectibleItemDetailsRoute(collectibleId)` | `collectibleItemDetailsScreen()` | Inline factory |
| **Add From Catalog** | `AddFromCatalogRoute` (data object) | `addFromCatalogScreen()` | `AddFromCatalogViewModel` |
| **Add To Notes** | `AddNotesRoute(collectibleId)` | `addToNotesScreen()` | Inline factory |
| **Image Gallery** | `ImageGalleryRoute(photoUrls, currentPhotoIndex, isActionTitleVisible)` | `imageGallery()` | Inline factory |
| **Create / Rename Folder** | `CreateRenameFolderRoute(mode)` | `createRenameFolder()` | Inline factory |
| **Collectible Folder** | `CollectibleFolderScreenRoute(folderId, folderName)` | `collectibleFolderScreen()` | Inline factory |
| **Notional Type Selection** | `NotionalTypeSelectionRoute(selectionResultKey)` | `notionalTypeSelectionScreen()` | Inline factory |
| **Add Collectibles to Folder** | `AddCollectiblesToFolderRoute()` | `addCollectiblesToFolderScreen()` | Inline factory |
| **Folder Selection** | `FolderSelectionRoute(collectibleIds)` | `folderSelectionScreen()` | Inline factory |
| **Manual Add / Edit** | `ManualAddEditCollectibleRoute(collectibleId?)` | `manualAddEditCollectibleScreen()` | `AddCollectibleViewModel` |
| **Archive Collectible** | `ArchiveCollectibleRoute(collectibleIds)` | `archiveCollectibleScreen()` | `ArchiveCollectibleViewModel` |
| **Repacks** | `RepacksRoute` (data object) | `repacksScreen()` | `RepacksViewModel` |
| **Parallel Details** | `ParallelDetailsRoute(...)` | `parallelDetails()` | Inline factory |

### CITA (Card Scan) Routes

These are in a separate navigation graph under `cashInTheAttic/`:

| Screen | Route |
|---|---|
| **Card Search Initialization** | `CardSearchInitializationRoute` (data object) |
| **Card Search List Match** | `CardSearchListMatchRoute(...)` |
| **Card Search Best Match** | `CardSearchBestMatchRoute` (data object) |
| **Card Search Similar Prices** | `CardSearchSimilarPricesRoute(...)` |
| **Card Search No Match** | `CardSearchNoMatchRoute` (data object) |
| **Card Search Item Details** | `CardSearchItemDetailsRoute` (data object) |

---

## Type-Safe Routes

Routes are `@Serializable` data classes or data objects. Example:

```kotlin
@Serializable
data class CollectibleItemDetailsRoute(val collectibleId: String) : CollectiblesRoute

@Serializable
data object AddFromCatalogRoute : CollectiblesRoute
```

Navigation file pattern:

```kotlin
fun NavGraphBuilder.myScreen() {
    composable<MyRoute>(
        enterTransition = { slideIn() },
        exitTransition = { slideOut() }
    ) { backStackEntry ->
        val viewModel = viewModel<MyViewModel>(factory = ...)
        MyScreen(viewModel = viewModel, onNavigate = { ... })
    }
}
```

---

## Deep Linking

| Deep Link | Processor | Handler |
|---|---|---|
| `ebay://.../citacardscan` | `CitaCardScanDeepLinkProcessor` | Launches CITA scan flow |
| `ebay://.../repacks` | `RepacksLinkProcessor` | Navigates to repacks |
| `ebay://.../vault` | Vault deep link | Navigates to vault submission list |

---

## Back Stack Management

| Pattern | Usage |
|---|---|
| `withRefresh()` | Passed to child screens to trigger data refresh on pop |
| `popBackStack()` | Standard navigation back |
| `savedStateHandle` | Retrieves data from previous screen (e.g., selected folder) |

---

## External Navigation

`DigitalCollectionsNavigationTarget` handles incoming navigation from other features (see [Architecture.md](Architecture.md#actionnavigationtarget)). It:

1. Extracts the page identifier from navigation params
2. Passes remaining params to `DigitalCollectionsFactory.buildActivityIntent()`
3. Handles modal vs regular launch via `KEY_IS_MODAL` param
