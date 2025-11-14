# Location Services

## FusedLocationProvider

```kotlin
val fusedLocationClient = LocationServices.getFusedLocationProviderClient(context)

fusedLocationClient.lastLocation.addOnSuccessListener { location ->
    location?.let {
        val lat = it.latitude
        val lng = it.longitude
    }
}
```

## LocationRequest

```kotlin
val locationRequest = LocationRequest.create().apply {
    interval = 10000
    fastestInterval = 5000
    priority = LocationRequest.PRIORITY_HIGH_ACCURACY
}

fusedLocationClient.requestLocationUpdates(
    locationRequest,
    locationCallback,
    Looper.getMainLooper()
)
```

## Recursos
- [Location](https://developer.android.com/training/location)
