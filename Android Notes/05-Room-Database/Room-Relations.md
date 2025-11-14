# Room Relations

## One-to-Many

```kotlin
@Entity
data class User(
    @PrimaryKey val userId: Int,
    val name: String
)

@Entity
data class Post(
    @PrimaryKey val postId: Int,
    val userId: Int,
    val title: String
)

data class UserWithPosts(
    @Embedded val user: User,
    @Relation(
        parentColumn = "userId",
        entityColumn = "userId"
    )
    val posts: List<Post>
)

@Dao
interface UserDao {
    @Transaction
    @Query("SELECT * FROM User")
    fun getUsersWithPosts(): Flow<List<UserWithPosts>>
}
```

## Many-to-Many

```kotlin
@Entity(primaryKeys = ["userId", "playlistId"])
data class UserPlaylistCrossRef(
    val userId: Int,
    val playlistId: Int
)

data class UserWithPlaylists(
    @Embedded val user: User,
    @Relation(
        parentColumn = "userId",
        entityColumn = "playlistId",
        associateBy = Junction(UserPlaylistCrossRef::class)
    )
    val playlists: List<Playlist>
)
```

## Recursos
- [Room Relations](https://developer.android.com/training/data-storage/room/relationships)
