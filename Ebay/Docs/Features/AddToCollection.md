# Add To Collection

The entry point for adding items to a collection. Users choose from four methods via the **Add Item Options** bottom sheet.

---

## Entry Point Comparison

| Method | Best For | Flow |
|---|---|---|
| **Add From Catalog** | Known items (name/search) | Search → Select Variant → Add |
| **Manual Add** | Items not in catalog | Form → Dynamic Fields → Submit |
| **Past Purchases** | Already bought on eBay | Select from purchase history → Add |
| **Scan (CITA)** | Physical cards with camera | Scan → Identify → Price → Add |

---

## Add Item Options Bottom Sheet

Triggered from the collection hub FAB or item details screen. Managed by `AddItemOptionsViewModel`.

**Tracking:** `COLLECTIBLES_ADD_CHOICE` (PageId 3515176), `COLLECTIBLES_ADD_CHOICE_MODULE` (ModuleId 67697)

---

## Feature Docs

- [Add From Catalog](AddFromCatalog.md) — Catalog search flow
- [Manual Add / Edit](ManualAddEdit.md) — Manual form
- [Cash In The Attic](CashInTheAttic.md) — Card scanning flow

Past purchases uses the `CollectiblesPastPurchaseRepository` (GraphQL) and shares the catalog search flow for adding.
