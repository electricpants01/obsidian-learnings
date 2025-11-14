# Firebase Cloud Messaging (FCM)

## Setup

```kotlin
dependencies {
    implementation("com.google.firebase:firebase-messaging-ktx")
}
```

## Service

```kotlin
class MyFirebaseMessagingService : FirebaseMessagingService() {
    override fun onMessageReceived(remoteMessage: RemoteMessage) {
        remoteMessage.notification?.let {
            showNotification(it.title, it.body)
        }
    }
    
    override fun onNewToken(token: String) {
        // Send token to server
    }
}
```

## Get Token

```kotlin
FirebaseMessaging.getInstance().token.addOnCompleteListener { task ->
    if (task.isSuccessful) {
        val token = task.result
    }
}
```

## Recursos
- [FCM](https://firebase.google.com/docs/cloud-messaging/android/client)
