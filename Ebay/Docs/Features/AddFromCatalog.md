# Add From Catalog

Search the catalog for a known collectible item and add it to a collection (Route: `AddFromCatalogRoute` data object).

## Flow

```
Search box (type-ahead) → Results list → Select variant (grading, parallel)
  → Dynamic form (server-driven fields) → Confirm → Add to collection
```

## Key Components

| Component | Description |
|---|---|
| `AddFromCatalogScreen` | Search UI with results grid/list |
| `AddFromCatalogViewModel` | State management (search, selection, form) |
| `AddFromCatalogRepository` | GraphQL catalog search data source |

## Data Sources

- **GraphQL:** Catalog search queries
- **GraphQL:** Category taxonomy for filtering
- **GraphQL:** Graded card options (when `collectibles.addFromCatalogGrading` is enabled)

## Dynamic Fields

Form fields after selecting a catalog item are driven by server response (notional type, aspects, attributes). This is shared with the [Manual Add](ManualAddEdit.md) flow.

## Folder Integration

When `collectibles.addToFolderFromCatalogSearch` is enabled, users can select a destination folder during the add flow.

## Tracking

- **PageId:** `COLLECTIBLES_ADD_FORM` (3515175)
- **ModuleId:** `COLLECTIBLES_ADD_FORM_MODULE` (94908)
- **Feature toggle:** `collectibles.addUsingCatalog`
