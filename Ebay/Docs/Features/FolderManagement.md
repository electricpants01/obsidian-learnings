# Folder Management

Organize collectible items into named folders for better organization.

## Operations

| Operation | Route | Description |
|---|---|---|
| **Create** | `CreateRenameFolderRoute(mode=Create)` | New folder name via bottom sheet |
| **Rename** | `CreateRenameFolderRoute(mode=Rename)` | Edit existing folder name |
| **Delete** | (inline action) | Confirm and remove folder |
| **Select** | `FolderSelectionRoute(collectibleIds)` | Multi-select items to add to folder |
| **Add to Folder** | `AddCollectiblesToFolderRoute()` | Bulk add items to selected folder |

## Folder Selection Screen

Presents a list of user folders for selecting a destination. Supports:

- Single-select (move one item)
- Multi-select (bulk add multiple items)
- Create new folder inline

## Data Source

- `CollectibleFolderRepository` (GraphQL) — all folder CRUD operations
- Folder-item associations are managed server-side

## Limits

Folder limits are enforced server-side. When a limit is hit, an error state is shown to the user.

## Feature Toggles

- `collectibles.folderSorting` — Sort folders on the hub screen
- `collectibles.folderInclusionInfo` — Show folder inclusion info
- `collectibles.addToFolderFromCatalogSearch` — Folder selection in catalog flow

## Tracking

- `FolderSelectionTrackingAssets` — dedicated tracking for folder selection
- `COLLECTIBLES_CATEGORY_OR_FOLDER` (PageId 3515254)
