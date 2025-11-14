# DataStore

## Setup

```kotlin
dependencies {
    implementation("androidx.datastore:datastore-preferences:1.0.0")
}
```

## Preferences DataStore

```kotlin
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")

class SettingsRepository(private val dataStore: DataStore<Preferences>) {
    private val THEME_KEY = stringPreferencesKey("theme")
    
    val theme: Flow<String> = dataStore.data.map { prefs ->
        prefs[THEME_KEY] ?: "light"
    }
    
    suspend fun saveTheme(theme: String) {
        dataStore.edit { prefs ->
            prefs[THEME_KEY] = theme
        }
    }
}
```

## Proto DataStore

```kotlin
val Context.userPrefsDataStore: DataStore<UserPreferences> by dataStore(
    fileName = "user_prefs.pb",
    serializer = UserPreferencesSerializer
)
```

## Recursos
- [DataStore](https://developer.android.com/topic/libraries/architecture/datastore)
