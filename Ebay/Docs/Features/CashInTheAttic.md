# Cash In The Attic (CITA)

Camera-based card scanning flow: **Scan → Identify → Price → Add to Collection**.

For the full technical documentation, see [CITA README](../digitalCollectionsImpl/src/main/java/com/ebay/mobile/digitalcollections/impl/cashInTheAttic/README.md).

---

## Flow Summary

```
User captures card image → Card identification (best match / list match / no match)
  → Show pricing / market data → User confirms → Added to collection
```

## Entry Points

| Entry Point | Mechanism |
|---|---|
| Collection hub add button | `CitaCtaOverride` |
| Deep link (`ebay://.../citacardscan`) | `CitaCardScanDeepLinkProcessor` |
| CITA entry point toggle | `collectibles.citaCollectionEntryPoint` |

## Confidence Flows

| Match Type | UX |
|---|---|
| **High (Best Match)** | Auto-select card, show pricing directly |
| **Low (List Match)** | Show candidate list for user to pick |
| **None (No Match)** | Show search/no-match options |

## Sub-Screens

- `CardSearchInitializationRoute` — Scanning UI
- `CardSearchBestMatchRoute` — Auto-matched card
- `CardSearchListMatchRoute` — Candidate list
- `CardSearchNoMatchRoute` — Manual search fallback
- `CardSearchItemDetailsRoute` — Card details before add
- `CardSearchSimilarPricesRoute` — Price comparison

## Key Dependencies

- **SmartLens** or **CashInTheAttic** — Visual identification (gated by `collectibles.useSmartLens`)
- **IIHUB API** — Image identification
- **Possession API** — Card data
- **UnifiedPriceGuidance** — Pricing
- **PSA Data** — Graded card info

## Feature Toggles

- `collectibles.stcConfigurationForCITA` — Sports Trading Cards
- `collectibles.ccgConfigurationForCITA` — Collectible Card Games
- `collectibles.citaCollectionEntryPoint` — Entry from collection
- `collectibles.cashInTheAtticDeepLink` — Deep link support
- `collectibles.useSmartLens` — Use SmartLens vs CITA

## Tracking

- **PageIds:** `COLLECTIBLES_ADD_IMAGE_SCAN` (4379093), `COLLECTIBLES_MATCHES_FOUND` (4379438), `COLLECTIBLES_NO_MATCHES_FOUND` (4379439), `COLLECTIBLES_REVIEW_SCANNED_CARDS` (4379447)
- **Event family:** `COLLECTION`
