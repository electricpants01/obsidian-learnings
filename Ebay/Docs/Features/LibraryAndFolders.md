# Library & Folders

## TabHost (Main Hub)

The main landing screen after entering Digital Collections. `TabHostRoute` (data object) in `view/tabHost/`.

**Tabs:** My Collection, Folders, Updates (configurable)

**ViewModel:** `TabHostViewModel` — manages tab state, header data, navigation events

**Key Components:**
- Collection summary header (total items, estimated value)
- Tab row for switching between views
- Add item FAB

---

## Collectible Folder Screen

Displays items in a specific folder (`CollectibleFolderScreenRoute`).

**Features:**
- Grid / list view toggle
- Sorting (when `collectibles.folderSorting` toggle is enabled)
- Bulk actions (when `collectibles.bulkActions` toggle is enabled)
- Folder info (when `collectibles.folderInclusionInfo` toggle is enabled)

**Data Sources:**
- `CollectibleFolderRepository` (GraphQL) — paginated items in a folder
- Collection summary (GraphQL)

**Tracking:** `COLLECTIBLES_CATEGORY_OR_FOLDER` (PageId 3515254), `COLLECTIBLES_INVENTORY_GRID` (ModuleId 67112), `COLLECTIBLES_LIBRARY` (ModuleId 67111)
