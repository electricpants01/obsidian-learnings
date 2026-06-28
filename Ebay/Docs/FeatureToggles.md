# Feature Toggles

All toggles are defined in `DigitalCollectionsFeatureToggles.kt`. They are managed via `ToggleRouter` from `:featureToggles`.

---

## Boolean Toggles (Default: false)

| Toggle Key | Default | Description | Feature |
|---|---|---|---|
| `collectibles.addUsingCatalog` | false | Enables adding collection item through catalog | Add From Catalog |
| `myebay.digitalCollections` | **true** | Enables digital collections (menu item + feature) | All |
| `myebay.vaultMenu` | false | Enables Vault (menu item + feature) | Vault |
| `myebay.priceGuidanceExcludeSales` | false | Match threshold for card detection | Price Guidance |
| `myebay.priceGuidanceFilterPanel` | false | Enables the Price Guidance expanded Filter Panel | Price Guidance |
| `collectibles.bulkActions` | false | Enables bulk actions in collectible lists | Library |
| `collectibles.useLocalSplashScreen` | false | Enables the local splash bottom sheet | Splash |
| `collectibles.psaPopReport` | false | Enables the PSA Pop Report on item details | Graded Cards |
| `collectibles.manualFormGQL` | false | Enables redesigned UI for manual add | Manual Add |
| `collectibles.upgByEpid` | false | Enables querying for price guidance by EPID | Price Guidance |
| `collectibles.addFromCatalogGrading` | false | Enables grading in the Add From Catalog flow | Add From Catalog |
| `collectibles.stcConfigurationForCITA` | false | Enables Sports Trading Card config for CITA | CITA |
| `collectibles.ccgConfigurationForCITA` | false | Enables Collectible Card Game config for CITA | CITA |
| `collectibles.addToFolderFromCatalogSearch` | false | Enables Add To Folder from Catalog Search | Folder |
| `collectibles.folderSorting` | false | Enables sorting the Folders tab | Folder |
| `collectibles.folderInclusionInfo` | false | Enables Folder Inclusion information | Folder |
| `collectibles.priceGuidanceActiveListings` | false | Enables listings type button and UPG flows | Price Guidance |
| `collectibles.citaCollectionEntryPoint` | false | Enables CITA flow from Collection | CITA |
| `collectibles.cashInTheAtticDeepLink` | false | Enables deep linking for CITA | CITA |
| `collectibles.activeListingsGQL` | false | Enables GQL-backed active listings | Price Guidance |
| `collectibles.priceAcrossParallelsCashInTheAttic` | false | Enables price across parallels for CITA | Parallels |
| `collectibles.priceAcrossParallelsViewItem` | false | Enables price across parallels for View Item | Parallels |
| `collectibles.priceAcrossParallelsMyCollection` | false | Enables price across parallels for My Collection | Parallels |
| `collectibles.repacksDiscovery` | false | Enables the Repacks Landing page | Repacks |
| `collectibles.upgComponents` | false | Enables new unified price guidance components | Price Guidance |
| `collectibles.bestOfferStrikethrough` | false | Enables showing best offer in Market Data | Price Guidance |
| `collectibles.draftGraphQLMigration` | false | Enables GraphQL mutations for draft CRUD | Manual Add |
| `collectibles.externalDataSoldGoldinViewItem` | false | Goldin external sold data on View Item | Price Guidance |
| `collectibles.externalDataSoldGoldinScanCards` | false | Goldin external sold data on Scan Cards | Price Guidance |
| `collectibles.externalDataSoldGoldinMyCollection` | false | Goldin external sold data on My Collection | Price Guidance |
| `collectibles.externalDataSoldTcgPlayerViewItem` | false | TCGPlayer external sold data on View Item | Price Guidance |
| `collectibles.externalDataSoldTcgPlayerScanCards` | false | TCGPlayer external sold data on Scan Cards | Price Guidance |
| `collectibles.externalDataSoldTcgPlayerMyCollection` | false | TCGPlayer external sold data on My Collection | Price Guidance |
| `collectibles.useSmartLens` | false | Use SmartLens for visual identification; otherwise CITA | Scanning |

---

## Integer Toggle

| Toggle Key | Default | Description |
|---|---|---|
| `myebay.digitalCollectionsSeekSurveyDaysBetweenNudges` | **183** | Days between seek survey nudges |

---

## URL Toggles

| Toggle Key | Production URL | QA URL | Purpose |
|---|---|---|---|
| `myebay.digitalCollectionsUrl` | `https://api.ebay.com/experience/collectibles/v1` | `https://apima.qa.ebay.com/experience/collectibles/v1` | Digital Collections API base |
| `myebay.priceGuidanceUrl` | `https://api.ebay.com/experience/price_guidance/v1` | `https://apima.qa.ebay.com/experience/price_guidance/v1` | Price Guidance API base |
| `collectibles.repacksContentManagementUrl` | `https://api.ebay.com/cdpcontent/repacks-landing` | `https://cdpcontent.vip.qa.ebay.com/cdpcontent/repacks-landing` | Repacks content management |

---

## How to Add a New Toggle

1. Add a `const val` in `DigitalCollectionsFeatureToggles` with a unique toggle key
2. Use it with `toggleRouter.isEnabled(MyToggle)` in the appropriate code
3. Register in the test module's `TestToggleRouter` if needed

## Testing with Toggles

Use `TestToggleRouter` in tests to override specific toggles:

```kotlin
val testToggleRouter = TestToggleRouter(
    mapOf(DigitalCollectionsFeatureToggles.MY_TOGGLE to true)
)
```
