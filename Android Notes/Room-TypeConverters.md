# Room TypeConverters

## Converter Básico

```kotlin
class Converters {
    @TypeConverter
    fun fromTimestamp(value: Long?): Date? {
        return value?.let { Date(it) }
    }
    
    @TypeConverter
    fun dateToTimestamp(date: Date?): Long? {
        return date?.time
    }
    
    @TypeConverter
    fun fromStringList(value: String): List<String> {
        return value.split(",")
    }
    
    @TypeConverter
    fun toStringList(list: List<String>): String {
        return list.joinToString(",")
    }
}

@Database(entities = [User::class], version = 1)
@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase()
```

## JSON Converter

```kotlin
class JsonConverters {
    private val json = Json { ignoreUnknownKeys = true }
    
    @TypeConverter
    fun fromAddress(address: Address): String {
        return json.encodeToString(address)
    }
    
    @TypeConverter
    fun toAddress(json: String): Address {
        return this.json.decodeFromString(json)
    }
}
```

## Recursos
- [TypeConverters](https://developer.android.com/training/data-storage/room/referencing-data)
