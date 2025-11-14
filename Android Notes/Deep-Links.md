# Deep Links

## Manifest

```xml
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" android:host="details" />
    </intent-filter>
</activity>
```

## Handle

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        intent?.data?.let { uri ->
            // myapp://details/123
            val id = uri.lastPathSegment
        }
    }
}
```

## Compose Navigation

```kotlin
composable(
    route = "details/{id}",
    deepLinks = listOf(
        navDeepLink { uriPattern = "myapp://details/{id}" }
    )
) { backStackEntry ->
    val id = backStackEntry.arguments?.getString("id")
    DetailsScreen(id)
}
```

## Recursos
- [Deep Links](https://developer.android.com/training/app-links/deep-linking)
