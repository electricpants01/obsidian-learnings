# Android TV

## Leanback Library

```kotlin
dependencies {
    implementation("androidx.leanback:leanback:1.2.0-alpha04")
}
```

## TV Activity

```kotlin
class MainActivity : FragmentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                .replace(R.id.main_browse_fragment, MainFragment())
                .commit()
        }
    }
}
```

## BrowseFragment

```kotlin
class MainFragment : BrowseSupportFragment() {
    override fun onActivityCreated(savedInstanceState: Bundle?) {
        super.onActivityCreated(savedInstanceState)
        setupUIElements()
        loadRows()
    }
}
```

## Recursos
- [Android TV](https://developer.android.com/training/tv)
