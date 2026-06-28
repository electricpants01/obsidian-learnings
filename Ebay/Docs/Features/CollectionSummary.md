# Collection Summary

Overview stats for the user's collection — shown in the hub header and various screens.

## ViewModel

`CollectionSummaryViewModel` manages:
- Total item count
- Estimated collection value
- Recent additions / changes
- Folder counts

## Data Sources

- `CollectionSummaryRepositoryImpl` (GraphQL) — aggregation queries for collection stats
- Reactive updates based on collection changes (add, remove, archive)

## Display

| Component | Location |
|---|---|
| Summary card | Hub header, folder screen header |
| Stats bar | Collection summary section |
| Recent activity | Updates tab |

## Tracking

- `COLLECTIBLES_SUMMARY_FOOTER` (ModuleId 106596)
- `COLLECTIBLES_MARKET_VALUE` (ModuleId 67117)
