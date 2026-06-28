# Archive & Mark As Sold

Remove items from active collection view by archiving or marking as sold (Route: `ArchiveCollectibleRoute(collectibleIds)`).

## Screens

| Screen | Route | Purpose |
|---|---|---|
| Archive Collectible | `ArchiveCollectibleRoute(collectibleIds)` | Archive one or multiple items |

## Flow

```
Select item(s) → Choose archive/sold → Confirm → Item removed from active collection
```

## Key Components

| Component | Description |
|---|---|
| `ArchiveCollectibleSeedData` | Holds item data for multi-item archive |
| `ArchiveCollectibleViewModel` | State management for archive operation |
| `ArchiveCollectibleScreen` | Archive confirmation UI |

## Use Cases

- `ArchiveOperationUseCase` — Archive item(s)
- `UnarchiveOperationUseCase` — Restore archived item(s)

## Data Sources

- `ArchiveCollectibleDataSource` (GraphQL) — Archive/unarchive mutations

## Impact

After archiving:
- Item moves from active to archived state
- Collection summary recalculates
- Archived items can be viewed via a filter

## Tracking

- **PageId:** `COLLECTIBLES_ARCHIVE_FORM` (4474454)
- **ModuleId:** `COLLECTIBLES_ARCHIVE_COLLECTIBLE_MODULE` (144687)
