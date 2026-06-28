# Tracking / Analytics

All tracking constants are centralized in `DcTrackingAsset.kt` in the root API module (`digitalCollections/src/main/java/.../tracking/DcTrackingAsset.kt`).

---

## Key Concepts

| Concept | Description | Example |
|---|---|---|
| **EventFamily** | Top-level tracking category | `COLLECTION`, `VAULT`, `PRICEGUIDE` |
| **PageId** | Unique identifier per screen/event | `COLLECTIBLES_HUB = 3685063` |
| **ModuleId** | Identifier for sub-sections within a page | `COLLECTIBLES_HUB_TABS = 161852` |
| **LinkId** | Identifier for individual UI actions | (varies per feature) |
| **EventProperty** | Key-value pairs attached to events | `notional_type_id`, `tabClickTag` |

### Event Families

| Constant | Value |
|---|---|
| `COLLECTION` | `"COLLECTION"` |
| `VAULT` | `"VAULT"` |
| `PRICE_GUIDANCE` | `"PRICEGUIDE"` |

---

## Per-Feature Tracking Pattern

Each feature defines tracking in three parts (interface + assets + implementation):

```kotlin
// 1. Interface
interface AddFromCatalogTracking {
    fun trackAddItemFromCatalogClicked()
    fun trackCatalogSearchSubmitted(query: String)
}

// 2. Assets class (constants)
class AddFromCatalogTrackingAssets {
    object PageIds {
        const val ADD_FROM_CATALOG_SCREEN = 4609138
    }
    object LinkIds {
        const val RESULT_ADD_TO_COLLECTION = 189124
    }
}

// 3. Implementation
class AddFromCatalogTrackingImpl @Inject constructor(
    private val tracker: Tracker
) : AddFromCatalogTracking {
    override fun trackAddItemFromCatalogClicked() {
        tracker.track(ClickEvent(PageIds.ADD_FROM_CATALOG_SCREEN, LinkIds.ADD_TO_COLLECTION))
    }
}
```

---

## PageIds Reference

| Constant | Value | Screen |
|---|---|---|
| `COLLECTIBLES_HUB` | `3685063` | Main collection hub |
| `COLLECTIBLES_ADD_CHOICE` | `3515176` | Add item option picker |
| `COLLECTIBLES_ADD_IMAGE_SCAN` | `4379093` | Image scan screen |
| `COLLECTIBLES_ADD_FORM` | `3515175` | Manual add / edit form |
| `COLLECTIBLES_NOTIONAL_TYPE_SELECTION` | `3515171` | Category selector |
| `COLLECTIBLES_ADD_CHOICE_PAST_PURCHASE` | `3515173` | Past purchases list |
| `COLLECTIBLES_PRICE_GUIDANCE` | `3641670` | Price guidance |
| `COLLECTIBLES_MATCHES_FOUND` | `4379438` | Scan matches found |
| `COLLECTIBLES_NO_MATCHES_FOUND` | `4379439` | Scan no matches |
| `COLLECTIBLES_REVIEW_SCANNED_CARDS` | `4379447` | Review scanned cards |
| `COLLECTIBLES_ARCHIVE_FORM` | `4474454` | Archive form |
| `COLLECTIBLES_ITEM_DETAILS` | `3685062` | Item details |
| `CARD_SEARCH_ITEM_DETAILS_SCREEN` | `4646611` | CITA item details |
| `COLLECTIBLES_CATEGORY_OR_FOLDER` | `3515254` | Category / folder view |
| `COLLECTIBLES_GLOBAL_SEARCH` | `3685064` | Global search |
| `COLLECTIBLES_QUICK_EDIT_PRICE_AND_DATE` | `4614823` | Quick edit price |
| `COLLECTIBLES_QUICK_EDIT_CONDITION` | `4612050` | Quick edit condition |
| `EDIT_PSA_CERT_NUMBER_DIALOG` | `4635693` | PSA cert entry |
| `PSA_POP_REPORT_DIALOG` | `4635969` | PSA Pop Report |
| `PG_UNIFIED_MODULE_PAGE_ID` | `4671005` | Unified Price Guidance |
| `CITA_SPLASH_SCREEN_PAGE_ID` | `4676084` | CITA splash |

### Vault PageIds

| Constant | Value |
|---|---|
| `VAULT_SUBMISSION_LIST` | `4436349` |
| `VAULT_SUBMISSION_DETAILS` | `4436350` |
| `VAULT_SUBMISSION_NOTE_FROM_AUTHENTICATOR` | `4443540` |
| `VAULT_SUBMISSION_PRINT_PACKING_SLIP` | `4440288` |
| `VAULT_CARD_HUB` | `4437283` |

---

## Adding New Tracking

1. Add your PageId / ModuleId / LinkId constants to `DcTrackingAsset`
2. Create a tracking interface for your feature
3. Create an `*Assets` class referencing the constants
4. Create an implementation using `Tracker`
5. Bind the implementation in the Dagger module and inject into your ViewModel
6. Call tracking methods from event handlers
