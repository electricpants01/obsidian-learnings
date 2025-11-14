# Bluetooth

## Permissions

```xml
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
```

## Scan Devices

```kotlin
val bluetoothAdapter = BluetoothAdapter.getDefaultAdapter()

if (bluetoothAdapter?.isEnabled == true) {
    bluetoothAdapter.startDiscovery()
}

val receiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        when (intent.action) {
            BluetoothDevice.ACTION_FOUND -> {
                val device = intent.getParcelableExtra<BluetoothDevice>(BluetoothDevice.EXTRA_DEVICE)
            }
        }
    }
}
```

## Recursos
- [Bluetooth](https://developer.android.com/guide/topics/connectivity/bluetooth)
