# Data Layer

## Data Source Types

| Type | Tech | Used For |
|---|---|---|
| **GraphQL** | Apollo GraphQL | Collectible details, folders, catalog search, CITA, collection summary |
| **REST (Experience Service)** | Retrofit | Legacy collectibles data, price guidance |
| **REST (COS)** | Retrofit | Checkout Orchestration Service |
| **Content Management** | CMS-backed | Repacks landing page content |
| **Preferences** | SharedPreferences | Feature toggle overrides, user preferences |

---

## Repository Pattern

Repositories follow Clean Architecture — interface in the `api/` package, implementation in `data/`:

```
digitalCollectionsImpl/src/main/java/.../impl/
├── api/
│   ├── CollectibleFolderRepository.kt          # Interface
│   ├── LegacyAddCollectibleRepository.kt        # Interface
│   ├── CollectiblesPastPurchaseRepository.kt    # Interface
│   ├── NotionalTypeRepository.kt                # Interface
│   ├── CardDisambiguationRepository.kt          # Interface
│   ├── collectibles/                            # ExpSvc interfaces
│   ├── datasource/                              # DataSource interfaces
│   └── graphql/                                 # GraphQL repository interfaces
├── data/
│   ├── CollectibleFolderRepositoryImpl.kt       # Implementation
│   ├── LegacyAddCollectibleRepositoryImpl.kt    # Implementation
│   ├── collectiblefolder/                       # Folder data
│   ├── collectibleItem/                         # Item data
│   ├── addfromcatalog/                          # Catalog search data
│   ├── cardSearch/                              # Card search data
│   ├── collectionSummary/                       # Summary data
│   └── price/                                   # Price data
```

---

## Key Repositories

| Repository | Data Source(s) | Purpose |
|---|---|---|
| `CollectibleFolderRepository` | GraphQL | Folder CRUD, folder-item associations |
| `CollectibleItemRepository` | GraphQL + Experience | Item details, updates |
| `LegacyAddCollectibleRepository` | GraphQL + COS | Add item form data, draft management |
| `AddFromCatalogRepository` | GraphQL | Catalog search results, graded card selection |
| `CollectiblesPastPurchaseRepository` | GraphQL | Past purchases list |
| `NotionalTypeRepository` | GraphQL | Notional type / category hierarchy |
| `CardDisambiguationRepositoryImpl` | GraphQL + IIHUB | Pre-fill listing candidates from image scan |
| `CollectionSummaryRepositoryImpl` | GraphQL | Collection statistics (total items, value) |
| `CardInsightsRepository` | GraphQL | Card insights / graded card data |
| `CollectionsRetrofitExpSvcDataSource` | REST (Experience) | Legacy experience service data |
| `PriceGuidanceRetrofitExpSvcDataSource` | REST (Experience) | Price guidance data |

---

## GraphQL Data Sources

Located in `api/graphql/`:

- `CollectiblesGraphQlDataSource`
- `AddCollectibleDataSource`
- `AddFromCatalogDataSource`
- `ArchiveCollectibleDataSource`
- `CardInsightsGraphQlDataSource`
- `CollectibleFolderRepositoryImpl` (direct GraphQL calls)
- `CollectionSummaryRepositoryImpl` (direct GraphQL calls)

---

## Business Models

Raw GraphQL/REST responses are transformed into business models in the `data/` packages. Convention:

```
Response (raw) → Transformer → Business Model (sealed interface / data class)
```

---

## Error Handling

The standard pattern uses sealed result types:

```kotlin
sealed class AsyncState<out T> {
    data object Loading : AsyncState<Nothing>()
    data class Success<T>(val data: T) : AsyncState<T>()
    data class Error(val message: String, val throwable: Throwable?) : AsyncState<Nothing>()
}
```

Each ViewModel exposes `StateFlow<AsyncState<UiState>>` consumed by the UI.
