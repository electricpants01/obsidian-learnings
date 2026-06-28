# Collectible Item Details

The main detail screen for viewing a single collectible item (Route: `CollectibleItemDetailsRoute(collectibleId)`).

## Sections

| Section | Description | Source |
|---|---|---|
| **Header** | Photo carousel, item title, price | `PhotoCarouselFactory` |
| **Attributes** | Key-value aspects (category, year, manufacturer, etc.) | GraphQL item query |
| **Condition** | Graded / ungraded condition display | `ConditionSelectionFactory` |
| **Price Guidance** | Market data, active/sold listings tabs | `UnifiedPriceGuidanceFactory` |
| **Notes** | User notes for the item | GraphQL |
| **Actions** | Edit, archive, share, add to folder | `COLLECTIBLES_ITEM_DETAILS_CTAS` module |

## Key Dependencies

- `CollectibleItemPriceGuidanceDataProvider` — Market data for this item
- `UpgConditionSelectionFactory` — Condition selector component
- `PhotoCarouselFactory` — Image carousel
- `ShowGradedCardFactory` — Card Insights (if graded)

## Data Sources

- GraphQL: `CollectibleDetailsQuery` for item metadata
- GraphQL: `UnifiedPriceGuidance` query for market data
- GraphQL: `CardItemCondition` query for graded card details
- REST (Experience Service): Price guidance via `PriceGuidanceRetrofitExpSvcDataSource`

## Tracking

- **PageId:** `COLLECTIBLES_ITEM_DETAILS` (3685062)
- **ModuleIds:** `COLLECTIBLES_MARKET_VALUE` (67117), `COLLECTIBLES_PRICE_GUIDANCE_STATS` (80818), `COLLECTIBLES_ITEM_DETAILS_CTAS` (144686)
- **Event families:** `COLLECTION`, `PRICEGUIDE`

## Feature Toggles

- `collectibles.psaPopReport` — PSA Pop Report tab
- `collectibles.priceGuidanceActiveListings` — Active listings tab
- `collectibles.upgComponents` — Unified Price Guidance components
- External sold data toggles (Goldin, TCGPlayer)
