# Repacks

Mystery pack / pack breaking feature — users can discover and purchase repacks of collectible items (Route: `RepacksRoute` data object).

## Screen

`RepacksScreen` displays repack listings and details. It currently uses its own internal Scaffold (see [Decor → ScaffoldDataBuilder Migration](../digitalCollectionsImpl/docs/DecorToScaffoldDataBuilderMigration.md) for migration plans).

## Data Sources

- **Repacks GraphQL** — Feature-specific queries
- **Content Management** — `RepacksContentManagementUrl` toggle for CMS-backed content via `:contentManagement`
- **Discovery Platform** — `RepacksDiscovery` toggle for Discovery-backed landing page

## ViewModel

`RepacksViewModel` manages:
- Repack listings state
- Content management data
- Navigation events

## Feature Toggles

- `collectibles.repacksDiscovery` — Discovery Platform-backed landing page
- `REPACKS_CONTENT_MANAGEMENT_URL` — URL toggle for CMS content

## Tracking

- `RepacksTrackingAssets` — dedicated repacks tracking constants
