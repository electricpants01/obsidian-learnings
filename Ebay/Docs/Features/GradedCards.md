# Graded Cards / Card Insights

Display graded collectible card information — grade, certification, grader details, PSA Pop Report.

## Module

This feature lives in its own sub-module: `digitalCollectionsGradedCard`.

## Key Components

| Component | Description |
|---|---|
| `ShowGradedCardFactory` | Self-contained component (see [Self-Contained Components](../SelfContainedComponents.md)) |
| `CardInsightsScreen` | Main card insights composable |
| `CardInsightsViewModel` | State management for card insights |
| `CardInsightsRepository` | GraphQL data source |

## Card Insights Sections

| Section | Composable | Description |
|---|---|---|
| **Summary** | `Summary` | Grade badge, card name, certification |
| **Additional Details** | `AdditionalDetails` | Grader, cert number, date graded |
| **Graders Row** | `GradersRow` | Logos for supported graders |
| **Population** | `Population` / `PopulationTable` | PSA Pop Report data |
| **Image Details** | `ImageDetails` | Card image with grade overlay |

## Supported Graders

- PSA (Professional Sports Authenticator)
- BGS / BVG (Beckett)
- SGC
- CGC

## Data Sources

| Data Source | Purpose |
|---|---|
| `CardInsightsGraphQlDataSource` | Main card insights data |
| `CardInsightsRepository` | GraphQL repository |
| `CgcDataTransformer` | CGC-specific data formatting |
| `PsaDataTransformer` | PSA-specific data formatting |

## Integration

`ShowGradedCardFactory` is consumed by:
- `CollectibleItemDetails` screen — shown in the condition/price section
- CITA card details — after scanning a graded card

## Feature Toggles

- `collectibles.psaPopReport` — PSA Pop Report tab on item details

## Tracking

- **GradedCardTelemetryEvent** — dedicated tracking events for card insights
- `CardInsightsTrackingHelper` — tracking helper within the graded card module
