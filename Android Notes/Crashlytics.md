# Firebase Crashlytics

## Setup

```kotlin
dependencies {
    implementation(platform("com.google.firebase:firebase-bom:32.7.0"))
    implementation("com.google.firebase:firebase-crashlytics-ktx")
}
```

## Log

```kotlin
val crashlytics = Firebase.crashlytics

// Log mensaje
crashlytics.log("User clicked button")

// Registrar excepción
try {
    // Code
} catch (e: Exception) {
    crashlytics.recordException(e)
}

// User ID
crashlytics.setUserId("user123")

// Custom keys
crashlytics.setCustomKey("screen", "home")
```

## Recursos
- [Crashlytics](https://firebase.google.com/docs/crashlytics/get-started?platform=android)
