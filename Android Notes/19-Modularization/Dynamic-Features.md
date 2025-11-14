# Dynamic Features

## On-Demand Delivery

```kotlin
val manager = SplitInstallManagerFactory.create(context)

val request = SplitInstallRequest.newBuilder()
    .addModule("dynamic_feature")
    .build()

manager.startInstall(request)
```

## Monitor Installation

```kotlin
manager.registerListener { state ->
    when (state.status()) {
        SplitInstallSessionStatus.DOWNLOADING -> {
            val progress = state.bytesDownloaded() / state.totalBytesToDownload()
        }
        SplitInstallSessionStatus.INSTALLED -> {
            // Feature ready
        }
    }
}
```

## Recursos
- [Dynamic Features](https://developer.android.com/guide/playcore/feature-delivery)
