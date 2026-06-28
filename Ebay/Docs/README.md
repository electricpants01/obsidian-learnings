# Digital Collections Documentation

## Overview

Digital Collections is an eBay feature that lets users manage their collectible items — view, add, organize, track value, scan cards, and more. It primarily targets **Sports Trading Cards (STC)** and **Collectible Card Games (CCG)**.

The module is split into **7 sub-modules**:

| Sub-module | Purpose |
|---|---|
| `digitalCollections` (root) | Public API module — interfaces, data models, tracking constants consumed by other modules |
| `digitalCollectionsImpl` | Core implementation — all screens, ViewModels, use cases, data sources, DI, navigation |
| `digitalCollectionsShared` | Shared UI composables — camera preview, bottom sheets, button bars |
| `digitalCollectionsGradedCard` | Graded card / Card Insights — PSA Pop Report, grader data |
| `digitalCollectionsApp` | App-level Dagger module wiring all sub-module DI together |
| `digitalCollectionsTestSupport` | Test helpers — shared POMs, Dagger test modules, paging test support |
| `digitalCollectionsInternalTestHelper` | Internal androidTest support — fakes, stubs, test rules |

---

## Documentation Index

### Getting Started
- [Overview](Overview.md) — High-level module description, entry points, capabilities
- [Architecture](Architecture.md) — Clean Architecture, UDF/MVI (CollectiblesScaffold), screen anatomy

### Cross-Cutting
- [Navigation](Navigation.md) — CollectiblesNavHost screens, type-safe routes, deep linking, back stack
- [Data Layer](DataLayer.md) — GraphQL + Retrofit data sources, repository pattern, caching
- [Feature Toggles](FeatureToggles.md) — All 37+ toggles with names, defaults, descriptions
- [Tracking](Tracking.md) — DcTrackingAsset framework, PageIds, ModuleIds, LinkIds, adding new tracking
- [Testing](Testing.md) — Unit test patterns, POM, fakes/turbine, Dagger test modules
- [Self-Contained Components](SelfContainedComponents.md) — Factory pattern for reusable UDF components

### Features

| Feature | Doc | Description |
|---|---|---|
| Library & Folders | [Features/LibraryAndFolders.md](Features/LibraryAndFolders.md) | Tab host, library grid/list, folder drill-down |
| Collectible Item Details | [Features/CollectibleItemDetails.md](Features/CollectibleItemDetails.md) | Item detail screen with all sections |
| Add To Collection | [Features/AddToCollection.md](Features/AddToCollection.md) | Entry point comparison (catalog, manual, past purchases, scan) |
| Add From Catalog | [Features/AddFromCatalog.md](Features/AddFromCatalog.md) | Catalog search & add flow |
| Manual Add / Edit | [Features/ManualAddEdit.md](Features/ManualAddEdit.md) | Manual form with dynamic server-driven fields |
| Folder Management | [Features/FolderManagement.md](Features/FolderManagement.md) | Folder CRUD, selection, bulk add to folder |
| Cash In The Attic (CITA) | [Features/CashInTheAttic.md](Features/CashInTheAttic.md) | Scan → Price → Action for card scanning |
| Graded Cards | [Features/GradedCards.md](Features/GradedCards.md) | Graded card display (PSA, BGS, etc.) |
| Repacks | [Features/Repacks.md](Features/Repacks.md) | Pack breaking feature |
| Parallel Details | [Features/ParallelDetails.md](Features/ParallelDetails.md) | Parallel card variants |
| Price Guidance | [Features/PriceGuidance.md](Features/PriceGuidance.md) | Market data, price insights, active listings |
| Collection Summary | [Features/CollectionSummary.md](Features/CollectionSummary.md) | Collection stats overview |
| Image Gallery | [Features/ImageGallery.md](Features/ImageGallery.md) | Full-screen photo viewer |
| Quick Edit | [Features/QuickEdit.md](Features/QuickEdit.md) | Inline editing via bottom sheet |
| Archive & Mark As Sold | [Features/ArchiveAndMarkAsSold.md](Features/ArchiveAndMarkAsSold.md) | Archive flow |

### Reference
- [Support](Support.md) — Slack channels, email DLs, PR process, tools

### Existing Docs (external)
- [Collectibles Architecture & Best Practices](../docs/CollectiblesArchitectureAndBestPractices.md) — Deep architecture guide
- [Decor → ScaffoldDataBuilder Migration](../digitalCollectionsImpl/docs/DecorToScaffoldDataBuilderMigration.md) — Migration plan
- [Cash In The Attic README](../digitalCollectionsImpl/src/main/java/com/ebay/mobile/digitalcollections/impl/cashInTheAttic/README.md) — Full CITA technical doc
- [Internal Test Helper README](../digitalCollectionsInternalTestHelper/README.md) — Test module docs

---

## Quick Links

- **Slack**: [#digital-collections-android](https://slack.com/app_redirect?channel=digital-collections-android)
- **Architecture**: [Architecture.md](Architecture.md)
- **Navigation**: [Navigation.md](Navigation.md)
- **Feature Toggles**: [FeatureToggles.md](FeatureToggles.md)
