# Hilt (Dependency Injection)

## Setup

```kotlin
// build.gradle (project)
plugins {
    id("com.google.dagger.hilt.android") version "2.50" apply false
}

// build.gradle (module)
plugins {
    id("com.google.dagger.hilt.android")
    id("com.google.devtools.ksp")
}

dependencies {
    implementation("com.google.dagger:hilt-android:2.50")
    ksp("com.google.dagger:hilt-compiler:2.50")
    implementation("androidx.hilt:hilt-navigation-compose:1.1.0")
}
```

## Application

```kotlin
@HiltAndroidApp
class MyApp : Application()
```

## Activity

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity()
```

## ViewModel

```kotlin
@HiltViewModel
class MyViewModel @Inject constructor(
    private val repository: Repository
) : ViewModel()
```

## Modules

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase {
        return Room.databaseBuilder(context, AppDatabase::class.java, "db").build()
    }
    
    @Provides
    fun provideUserDao(database: AppDatabase): UserDao {
        return database.userDao()
    }
}
```

## Recursos
- [Hilt](https://developer.android.com/training/dependency-injection/hilt-android)
