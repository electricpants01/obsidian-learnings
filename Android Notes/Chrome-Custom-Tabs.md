# Chrome Custom Tabs

## Setup

```kotlin
dependencies {
    implementation("androidx.browser:browser:1.7.0")
}
```

## Uso

```kotlin
val url = "https://www.example.com"
val customTabsIntent = CustomTabsIntent.Builder()
    .setToolbarColor(ContextCompat.getColor(context, R.color.primary))
    .setShowTitle(true)
    .build()

customTabsIntent.launchUrl(context, Uri.parse(url))
```

## Recursos
- [Custom Tabs](https://developer.chrome.com/docs/android/custom-tabs)
