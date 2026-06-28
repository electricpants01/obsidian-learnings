# Image Gallery

Full-screen photo viewer for collectible item images (Route: `ImageGalleryRoute(photoUrls, currentPhotoIndex, isActionTitleVisible)`).

## Components

| Component | Description |
|---|---|
| `ImageGalleryRoute` | Navigation route with photo URLs, current index, and visibility |
| `PhotoCarouselFactory` | Self-contained photo carousel component |

## Features

- Swipe left/right between photos
- Pinch-to-zoom
- Action bar visibility toggle

## Integration Points

| Screen | Trigger |
|---|---|
| Collectible Item Details | Tap thumbnail photo |
| Add From Catalog | Preview catalog images |
| Manual Add / Edit | Preview uploaded photos |
| CITA Card Details | Preview scanned card |

## Dependencies

- `PhotoCarouselFactory` — Image loading and display
- `:image` module — Compose image loading support

## Tracking

- Page impressions and click events for image gallery
