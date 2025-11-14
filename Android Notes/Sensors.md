# Sensors

## Sensor Manager

```kotlin
val sensorManager = getSystemService(Context.SENSOR_SERVICE) as SensorManager

val accelerometer = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER)
val gyroscope = sensorManager.getDefaultSensor(Sensor.TYPE_GYROSCOPE)

val sensorListener = object : SensorEventListener {
    override fun onSensorChanged(event: SensorEvent) {
        when (event.sensor.type) {
            Sensor.TYPE_ACCELEROMETER -> {
                val x = event.values[0]
                val y = event.values[1]
                val z = event.values[2]
            }
        }
    }
    
    override fun onAccuracyChanged(sensor: Sensor, accuracy: Int) {}
}

sensorManager.registerListener(
    sensorListener,
    accelerometer,
    SensorManager.SENSOR_DELAY_NORMAL
)
```

## Recursos
- [Sensors](https://developer.android.com/guide/topics/sensors/sensors_overview)
