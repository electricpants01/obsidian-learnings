# Quick Edit

Inline editing of collectible attributes via a bottom sheet, without navigating to the full edit screen.

## Editable Sections

| Section | FormField | PageId |
|---|---|---|
| **Price & Date** | Price, purchase date | `COLLECTIBLES_QUICK_EDIT_PRICE_AND_DATE` (4614823) |
| **Condition** | Condition selection | `COLLECTIBLES_QUICK_EDIT_CONDITION` (4612050) |
| **Notes** | Text notes | (uses add notes screen) |

## Flow

```
User taps edit icon on item → Bottom sheet appears with section form
  → User edits values → Save → State updates
```

## ViewModel

`QuickEditBottomSheetViewModel` manages:
- `InitState(section, collectibleId)` — Initializes with current values
- Edit state for each form section
- Save / cancel actions

## Integration

Triggered from:
- Collectible Item Details screen (edit buttons)
- Collectible Folder screen (bulk edit)

## Tracking

- **PageIds:** `COLLECTIBLES_QUICK_EDIT_PRICE_AND_DATE` (4614823), `COLLECTIBLES_QUICK_EDIT_CONDITION` (4612050)
