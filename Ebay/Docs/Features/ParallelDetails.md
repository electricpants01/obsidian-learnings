# Parallel Details

View parallel/variant versions of a collectible card — different grades, print runs, or variations (Route: `ParallelDetailsRoute`).

## Screen

`ParallelDetailsScreen` displays:
- Original item banner (navigation back to source item)
- Parallel variants grid/list
- Filter controls (grade, time period, sort)

## Configuration

`ParallelDetailsConfiguration` controls:
- Visible filter options
- Default sort order
- Data source selection

## Data Sources

- **GraphQL:** `ParallelDetailsQuery` for parallel variant data
- **UnifiedPriceGuidance:** Price comparisons across variants

## Use Cases

- `PriceAcrossParallels*` — Price comparisons (CITA, View Item, My Collection flows)

## Feature Toggles

- `collectibles.priceAcrossParallelsCashInTheAttic` — Parallels in CITA
- `collectibles.priceAcrossParallelsViewItem` — Parallels in View Item
- `collectibles.priceAcrossParallelsMyCollection` — Parallels in My Collection

## Tracking

- **PageId:** `PG_UNIFIED_MODULE_PAGE_ID` (4671005)
- `ParallelsTrackingAssets` — dedicated tracking with page impression properties
