# Services

## Foreground Service

```kotlin
class MyForegroundService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val notification = NotificationCompat.Builder(this, CHANNEL_ID)
            .setContentTitle("Service Running")
            .setSmallIcon(R.drawable.ic_service)
            .build()
        
        startForeground(1, notification)
        return START_STICKY
    }
    
    override fun onBind(intent: Intent?): IBinder? = null
}

// Iniciar
val intent = Intent(context, MyForegroundService::class.java)
ContextCompat.startForegroundService(context, intent)
```

## Bound Service

```kotlin
class MyBoundService : Service() {
    private val binder = LocalBinder()
    
    inner class LocalBinder : Binder() {
        fun getService(): MyBoundService = this@MyBoundService
    }
    
    override fun onBind(intent: Intent): IBinder = binder
}
```

## Recursos
- [Services](https://developer.android.com/guide/components/services)
