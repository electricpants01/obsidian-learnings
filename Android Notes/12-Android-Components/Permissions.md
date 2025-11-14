# Permissions

## Runtime Permissions

```kotlin
// Verificar permiso
if (ContextCompat.checkSelfPermission(context, Manifest.permission.CAMERA)
    == PackageManager.PERMISSION_GRANTED) {
    // Usar cámara
}

// Solicitar permiso
val requestPermissionLauncher = registerForActivityResult(
    ActivityResultContracts.RequestPermission()
) { isGranted ->
    if (isGranted) {
        // Permiso concedido
    }
}

requestPermissionLauncher.launch(Manifest.permission.CAMERA)
```

## Múltiples Permisos

```kotlin
val requestMultiplePermissions = registerForActivityResult(
    ActivityResultContracts.RequestMultiplePermissions()
) { permissions ->
    permissions.entries.forEach {
        Log.d("Permission", "${it.key} = ${it.value}")
    }
}

requestMultiplePermissions.launch(arrayOf(
    Manifest.permission.CAMERA,
    Manifest.permission.ACCESS_FINE_LOCATION
))
```

## Recursos
- [Permissions](https://developer.android.com/training/permissions)
