# Room Migration

## Migration Básica

```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("ALTER TABLE users ADD COLUMN age INTEGER NOT NULL DEFAULT 0")
    }
}

val db = Room.databaseBuilder(context, AppDatabase::class.java, "database")
    .addMigrations(MIGRATION_1_2)
    .build()
```

## Múltiples Migrations

```kotlin
val MIGRATION_2_3 = object : Migration(2, 3) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("CREATE TABLE posts (id INTEGER PRIMARY KEY NOT NULL, title TEXT NOT NULL)")
    }
}

Room.databaseBuilder(context, AppDatabase::class.java, "database")
    .addMigrations(MIGRATION_1_2, MIGRATION_2_3)
    .build()
```

## Fallback Destructive Migration

```kotlin
Room.databaseBuilder(context, AppDatabase::class.java, "database")
    .fallbackToDestructiveMigration()
    .build()
```

## Recursos
- [Room Migration](https://developer.android.com/training/data-storage/room/migrating-db-versions)
