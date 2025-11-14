# Logging

## Logcat

```kotlin
import android.util.Log

Log.v(TAG, "Verbose")
Log.d(TAG, "Debug")
Log.i(TAG, "Info")
Log.w(TAG, "Warning")
Log.e(TAG, "Error")
```

## Timber

```kotlin
dependencies {
    implementation("com.jakewharton.timber:timber:5.0.1")
}

// Application
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        if (BuildConfig.DEBUG) {
            Timber.plant(Timber.DebugTree())
        }
    }
}

// Uso
Timber.d("Debug message")
Timber.e(exception, "Error occurred")
```

## Recursos
- [Logging](https://developer.android.com/studio/debug/am-logcat)
