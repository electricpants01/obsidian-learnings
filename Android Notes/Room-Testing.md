# Room Testing

## Setup

```kotlin
dependencies {
    testImplementation("androidx.room:room-testing:2.6.1")
    testImplementation("androidx.test:core:1.5.0")
    testImplementation("junit:junit:4.13.2")
}
```

## Test Database

```kotlin
@RunWith(AndroidJUnit4::class)
class UserDaoTest {
    private lateinit var database: AppDatabase
    private lateinit var userDao: UserDao
    
    @Before
    fun setup() {
        database = Room.inMemoryDatabaseBuilder(
            ApplicationProvider.getApplicationContext(),
            AppDatabase::class.java
        ).allowMainThreadQueries().build()
        
        userDao = database.userDao()
    }
    
    @After
    fun tearDown() {
        database.close()
    }
    
    @Test
    fun insertAndGetUser() = runTest {
        val user = User(1, "John", "john@test.com")
        userDao.insert(user)
        
        val retrieved = userDao.getById(1)
        assertEquals(user, retrieved)
    }
}
```

## Recursos
- [Room Testing](https://developer.android.com/training/data-storage/room/testing-db)
