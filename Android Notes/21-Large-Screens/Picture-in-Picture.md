# Picture-in-Picture (PiP)

## Habilitar PiP

```xml
<activity
    android:name=".VideoActivity"
    android:supportsPictureInPicture="true"
    android:configChanges="screenSize|smallestScreenSize|screenLayout|orientation" />
```

## Entrar a PiP

```kotlin
val params = PictureInPictureParams.Builder()
    .setAspectRatio(Rational(16, 9))
    .build()

enterPictureInPictureMode(params)
```

## Detectar PiP

```kotlin
override fun onPictureInPictureModeChanged(
    isInPictureInPictureMode: Boolean,
    newConfig: Configuration
) {
    if (isInPictureInPictureMode) {
        // Hide UI
    } else {
        // Show UI
    }
}
```

## Recursos
- [PiP](https://developer.android.com/guide/topics/ui/picture-in-picture)
