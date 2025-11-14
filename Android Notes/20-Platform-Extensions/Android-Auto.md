# Android Auto

## Setup

```xml
<application>
    <meta-data
        android:name="com.google.android.gms.car.application"
        android:resource="@xml/automotive_app_desc" />
</application>
```

## Media Service

```kotlin
class MyMediaService : MediaBrowserServiceCompat() {
    override fun onGetRoot(
        clientPackageName: String,
        clientUid: Int,
        rootHints: Bundle?
    ): BrowserRoot {
        return BrowserRoot("root", null)
    }
    
    override fun onLoadChildren(
        parentId: String,
        result: Result<List<MediaBrowserCompat.MediaItem>>
    ) {
        // Return media items
    }
}
```

## Recursos
- [Android Auto](https://developer.android.com/training/cars)
