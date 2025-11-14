# Camera

## CameraX

```kotlin
dependencies {
    implementation("androidx.camera:camera-core:1.3.1")
    implementation("androidx.camera:camera-camera2:1.3.1")
    implementation("androidx.camera:camera-lifecycle:1.3.1")
    implementation("androidx.camera:camera-view:1.3.1")
}
```

## Preview

```kotlin
val cameraProviderFuture = ProcessCameraProvider.getInstance(context)

cameraProviderFuture.addListener({
    val cameraProvider = cameraProviderFuture.get()
    
    val preview = Preview.Builder().build()
    preview.setSurfaceProvider(previewView.surfaceProvider)
    
    val cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA
    
    cameraProvider.bindToLifecycle(
        lifecycleOwner,
        cameraSelector,
        preview
    )
}, ContextCompat.getMainExecutor(context))
```

## Capturar Imagen

```kotlin
val imageCapture = ImageCapture.Builder().build()

imageCapture.takePicture(
    ContextCompat.getMainExecutor(context),
    object : ImageCapture.OnImageCapturedCallback() {
        override fun onCaptureSuccess(image: ImageProxy) {
            // Process image
            image.close()
        }
    }
)
```

## Recursos
- [CameraX](https://developer.android.com/training/camerax)
