# Media Player

## ExoPlayer

```kotlin
dependencies {
    implementation("androidx.media3:media3-exoplayer:1.2.0")
    implementation("androidx.media3:media3-ui:1.2.0")
}
```

## Básico

```kotlin
val player = ExoPlayer.Builder(context).build()

val mediaItem = MediaItem.fromUri(videoUri)
player.setMediaItem(mediaItem)
player.prepare()
player.play()

// En Compose
@Composable
fun VideoPlayer(videoUri: Uri) {
    val context = LocalContext.current
    val player = remember {
        ExoPlayer.Builder(context).build().apply {
            setMediaItem(MediaItem.fromUri(videoUri))
            prepare()
        }
    }
    
    DisposableEffect(Unit) {
        onDispose { player.release() }
    }
    
    AndroidView(factory = { PlayerView(it).apply { this.player = player } })
}
```

## Recursos
- [ExoPlayer](https://developer.android.com/guide/topics/media/exoplayer)
