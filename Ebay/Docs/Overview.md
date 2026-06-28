# Digital Collections Overview

## What Is Digital Collections?

Digital Collections lets eBay users manage their physical collectible items in a digital catalog. Users can view their collection, add items (via catalog search, manual form, past purchases, or camera scan), organize them into folders, track market value, and manage their inventory.

### Supported Categories
- **Sports Trading Cards (STC)** — Baseball, basketball, football, soccer, etc.
- **Collectible Card Games (CCG)** — Pokémon, Magic: The Gathering, Yu-Gi-Oh!, etc.
- Other collectible categories supported through manual add

---

## Entry Points

Users reach Digital Collections through several paths:

| Entry Point | Destination | Implementation |
|---|---|---|
| My eBay tab | Collection hub | `CollectiblesContainer` → `TabHostScreen` |
| Deep link (scan card) | CITA scan flow | `CitaCardScanDeepLinkProcessor` |
| Deep link (repacks) | Repacks screen | `RepacksLinkProcessor` |
| View Item → Add to Collection | Add from catalog | `AddFromCatalogRoute` |
| Scan CTA on other screens | CITA initialization | `CitaDetailsProvider` |
| Vault menu | Vault submissions | `VaultSubmissionListRoute` |

---

## Key Capabilities

- **View Collection** — Grid/list view of items with filtering and sorting
- **Add Items** — Catalog search, manual form, past purchases, camera scan
- **Organize** — Folders with CRUD, bulk add, sorting
- **Track Value** — Price guidance with market data, active/sold listings, PSA Pop Report
- **Scan Cards** — Camera-based card identification (CITA via SmartLens or CashInTheAttic)
- **Graded Cards** — Card Insights for PSA, BGS, CGC, SGC graded cards
- **Repacks** — Mystery pack breaking and discovery
- **Quick Edit** — Inline editing of price, condition, notes
- **Archive/Sold** — Mark items as sold or archived

---

## Sub-Module Dependency Diagram

```
┌─────────────────────────────────────────────────────┐
│                   digitalCollections                 │
│   (public API: factories, models, tracking assets)   │
└──────────┬──────────────────────────────────────────┘
           │ depends on
┌──────────▼──────────────────────────────────────────┐
│                digitalCollectionsImpl                 │
│   (all screens, VMs, use cases, repos, data sources)  │
│   ┌────────────────────────────────────────────────┐ │
│   │  digitalCollectionsShared                       │ │
│   │  (shared composables: camera, bottom sheets)    │ │
│   └────────────────────────────────────────────────┘ │
└──────────┬──────────────────────────────────────────┘
           │
┌──────────▼──────────────────────────────────────────┐
│              digitalCollectionsApp                    │
│   (top-level Dagger module, wires all sub-module DI)  │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────┐
│              digitalCollectionsGradedCard              │
│   (standalone module for graded card / Card Insights)  │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────┐
│         digitalCollectionsTestSupport                  │
│   (POMs, Dagger test modules, paging test support)    │
└──────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────┐
│      digitalCollectionsInternalTestHelper             │
│   (androidTest fakes, stubs, test rules)              │
└──────────────────────────────────────────────────────┘
```

---

## Key Dependencies

| Domain | Modules |
|---|---|
| **Experience Service** | `:experience:experienceUx`, `:experience:experienceNavigation`, `:experience:experienceData` |
| **Navigation** | `:navigation` (ActionNavigationTarget), `:universalLink` (LinkProcessor) |
| **UPG (Price Guidance)** | `:businessComponent:businessComponentUpg*` |
| **Card Scanning** | `:cashInTheAttic`, `:smartLens` |
| **Camera** | `:camera`, `:cameraCapture` |
| **UI** | `:uiCompose`, `:ui`, `:uiShared`, `:platform:uiComposeTheme` |
| **Feature Toggles** | `:featureToggles` (ToggleRouter) |
| **Preferences** | `:preferences` (PreferencesRepository) |
| **Analytics** | `:telemetryLogger` |
| **Image Loading** | `:image` (Compose support) |
| **Discovery** | `:discovery`, `:discoveryUi` (repacks) |
| **Deep Linking** | `:deepLinking`, `:contentManagement` |
| **Network** | `:retrofit` (COS + Experience Service) |
