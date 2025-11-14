# Analytics

## Firebase Analytics

```kotlin
dependencies {
    implementation("com.google.firebase:firebase-analytics-ktx")
}
```

## Log Events

```kotlin
val analytics = Firebase.analytics

analytics.logEvent("button_click") {
    param("button_name", "submit")
    param("screen", "login")
}

analytics.setUserProperty("user_type", "premium")
```

## Recursos
- [Analytics](https://firebase.google.com/docs/analytics/get-started?platform=android)
