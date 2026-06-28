# Manual Add / Edit

Add items via a form when they aren't in the catalog, or edit existing items (Route: `ManualAddEditCollectibleRoute(collectibleId?)`).

## Form Sections

| Section | Description |
|---|---|
| **Required fields** | Title, category, condition |
| **Recommended fields** | Price, purchase date, notes |
| **Additional fields** | Server-driven dynamic fields based on notional type |
| **Photos** | Camera capture or gallery upload |

## Dynamic Form Rendering

Form fields are server-driven with these types:

- Text / Number input
- Dropdown / Picker
- Date picker
- Currency amount
- Checkbox
- Large text (notes)

Field visibility is controlled by groups that can be shown/hidden based on notional type selection.

## Photo Upload

- Uses `ImageUploadUseCase` for handling image capture and upload
- `PhotoEditorComposeViewModel` for photo editing (crop, rotate)
- Photos are stored as part of the collectible item

## Notional Type Selection

Selecting/changing the notional type (category) refreshes the form fields from the server. See `NotionalTypeSelectionRoute`.

## Multi-Add Mode

`ManualAddMode.MULTI_ADD` allows batch adding multiple items without leaving the form.

## ViewModel

`AddCollectibleViewModel` manages:
- Form state (all fields, validation, visibility groups)
- Photo state (captured images, upload progress)
- Draft state (auto-save, restore)
- Submission state (loading, success, error)

## Feature Toggles

- `collectibles.manualFormGQL` — Enables the redesigned GraphQL-backed form UI
- `collectibles.draftGraphQLMigration` — Enables GraphQL mutations for draft create/delete

## Tracking

- **PageId:** `COLLECTIBLES_ADD_FORM` (3515175)
- **ModuleId:** `COLLECTIBLES_ADD_FORM_MODULE` (94908)
