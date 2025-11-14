# Room Entity

## Entity Básica

```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val name: String,
    val email: String,
    val age: Int
)
```

## Anotaciones

```kotlin
@Entity(
    tableName = "users",
    indices = [Index(value = ["email"], unique = true)],
    foreignKeys = [
        ForeignKey(
            entity = Department::class,
            parentColumns = ["id"],
            childColumns = ["departmentId"],
            onDelete = ForeignKey.CASCADE
        )
    ]
)
data class User(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    
    @ColumnInfo(name = "user_name")
    val name: String,
    
    @Ignore
    val tempField: String? = null
)
```

## Embedded

```kotlin
data class Address(
    val street: String,
    val city: String,
    val zipCode: String
)

@Entity
data class User(
    @PrimaryKey val id: Int,
    val name: String,
    @Embedded val address: Address
)
```

## Recursos
- [Room Entities](https://developer.android.com/training/data-storage/room/defining-data)
