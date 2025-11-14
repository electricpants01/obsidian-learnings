# WorkManager

## Setup

```kotlin
dependencies {
    implementation("androidx.work:work-runtime-ktx:2.9.0")
}
```

## Worker

```kotlin
class SyncWorker(context: Context, params: WorkerParameters) : CoroutineWorker(context, params) {
    override suspend fun doWork(): Result {
        return try {
            repository.sync()
            Result.success()
        } catch (e: Exception) {
            Result.retry()
        }
    }
}
```

## Enqueue Work

```kotlin
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.CONNECTED)
    .setRequiresBatteryNotLow(true)
    .build()

val syncRequest = PeriodicWorkRequestBuilder<SyncWorker>(15, TimeUnit.MINUTES)
    .setConstraints(constraints)
    .build()

WorkManager.getInstance(context).enqueueUniquePeriodicWork(
    "sync",
    ExistingPeriodicWorkPolicy.KEEP,
    syncRequest
)
```

## Recursos
- [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager)
