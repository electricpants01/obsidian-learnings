# Price Guidance

Market data and price insights for collectible items, delivered via the **Unified Price Guidance (UPG)** component.

## Component

`UnifiedPriceGuidanceFactory` is a self-contained component (see [Self-Contained Components](../SelfContainedComponents.md)).

## Data Tabs

| Tab | Description | Toggle |
|---|---|---|
| **Active Listings** | Current eBay listings for this item | `collectibles.priceGuidanceActiveListings` |
| **Sold Listings** | Recently sold prices | Always shown |
| **Market Data** | Price trends and statistics | Always shown |
| **Price Guide** | Estimated value range | Always shown |
| **PSA Pop Report** | PSA population counts | `collectibles.psaPopReport` |

## External Sold Data

| Source | View Item | Scan Cards | My Collection |
|---|---|---|---|
| **Goldin** | `externalDataSoldGoldinViewItem` | `externalDataSoldGoldinScanCards` | `externalDataSoldGoldinMyCollection` |
| **TCGPlayer** | `externalDataSoldTcgPlayerViewItem` | `externalDataSoldTcgPlayerScanCards` | `externalDataSoldTcgPlayerMyCollection` |

## Data Sources

- **REST (Experience Service):** `PriceGuidanceRetrofitExpSvcDataSource`
- **GraphQL:** `UnifiedPriceGuidance` query (when `collectibles.upgByEpid` is enabled)
- **GraphQL:** Active listings (when `collectibles.activeListingsGQL` is enabled)
- **Business components:** `businessComponentUpg*` modules for chart data, pop reports, price insights

## Feature Toggles

- `myebay.priceGuidanceExcludeSales` — Match threshold
- `myebay.priceGuidanceFilterPanel` — Expanded filter panel
- `collectibles.upgComponents` — Use new UPG components
- `collectibles.bestOfferStrikethrough` — Show best offer in market data
- All external data sold toggles (see above)

## Tracking

- **PageId:** `COLLECTIBLES_PRICE_GUIDANCE` (3641670) or `PG_UNIFIED_MODULE_PAGE_ID` (4671005)
- **Event family:** `PRICEGUIDE`
