# Notifications

## Crear Notification Channel

```kotlin
val channel = NotificationChannel(
    "channel_id",
    "Channel Name",
    NotificationManager.IMPORTANCE_DEFAULT
)

val notificationManager = getSystemService(NotificationManager::class.java)
notificationManager.createNotificationChannel(channel)
```

## Mostrar Notification

```kotlin
val notification = NotificationCompat.Builder(context, "channel_id")
    .setContentTitle("Title")
    .setContentText("Content")
    .setSmallIcon(R.drawable.ic_notification)
    .setPriority(NotificationCompat.PRIORITY_DEFAULT)
    .build()

notificationManager.notify(1, notification)
```

## Recursos
- [Notifications](https://developer.android.com/develop/ui/views/notifications)
