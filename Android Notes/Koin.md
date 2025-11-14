# Koin

## Setup

```kotlin
dependencies {
    implementation("io.insert-koin:koin-android:3.5.3")
    implementation("io.insert-koin:koin-androidx-compose:3.5.3")
}
```

## Módulos

```kotlin
val appModule = module {
    single { AppDatabase.getDatabase(androidContext()) }
    single { get<AppDatabase>().userDao() }
    single { UserRepository(get()) }
    viewModel { UserViewModel(get()) }
}

class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidContext(this@MyApp)
            modules(appModule)
        }
    }
}
```

## Inyección

```kotlin
class UserViewModel(private val repository: UserRepository) : ViewModel()

// En Compose
@Composable
fun UserScreen(viewModel: UserViewModel = koinViewModel())
```

## Recursos
- [Koin](https://insert-koin.io/)
