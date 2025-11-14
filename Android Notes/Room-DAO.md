# Room DAO

## DAO Básico

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAll(): Flow<List<User>>
    
    @Query("SELECT * FROM users WHERE id = :userId")
    suspend fun getById(userId: Int): User?
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(user: User)
    
    @Insert
    suspend fun insertAll(users: List<User>)
    
    @Update
    suspend fun update(user: User)
    
    @Delete
    suspend fun delete(user: User)
    
    @Query("DELETE FROM users")
    suspend fun deleteAll()
}
```

## Query con parámetros

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users WHERE age > :minAge ORDER BY name ASC")
    fun getUsersOlderThan(minAge: Int): Flow<List<User>>
    
    @Query("SELECT * FROM users WHERE name LIKE '%' || :searchQuery || '%'")
    fun searchUsers(searchQuery: String): Flow<List<User>>
}
```

## Transacciones

```kotlin
@Dao
interface UserDao {
    @Transaction
    suspend fun updateUserAndPosts(user: User, posts: List<Post>) {
        update(user)
        insertPosts(posts)
    }
}
```

## Recursos
- [Room DAO](https://developer.android.com/training/data-storage/room/accessing-data)
