# Memory Leaks

## Detección con LeakCanary

```kotlin
dependencies {
    debugImplementation("com.squareup.leakcanary:leakcanary-android:2.13")
}
```

## Prevención

```kotlin
// ❌ MALO - leak de Activity
class MyViewModel : ViewModel() {
    var activity: Activity? = null // Leak!
}

// ✅ BUENO - usar Application Context
class MyViewModel(private val context: Context) : ViewModel()

// ❌ MALO - listener sin cleanup
class MyFragment : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        viewModel.data.observe(viewLifecycleOwner) { }
    }
}
```

## Recursos
- [Memory Leaks](https://developer.android.com/topic/performance/memory)
