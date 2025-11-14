# BroadcastReceiver

## Registrar Receiver

```kotlin
class MyReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        when (intent.action) {
            Intent.ACTION_BOOT_COMPLETED -> {
                // Handle boot completed
            }
        }
    }
}

// Manifest
<receiver android:name=".MyReceiver">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
</receiver>

// Dinámico
val receiver = MyReceiver()
val filter = IntentFilter(Intent.ACTION_BATTERY_CHANGED)
registerReceiver(receiver, filter)
```

## Recursos
- [BroadcastReceiver](https://developer.android.com/guide/components/broadcasts)
