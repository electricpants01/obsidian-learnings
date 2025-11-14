# Retrofit

## Setup

```kotlin
dependencies {
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    // o Kotlin Serialization
    implementation("com.jakewharton.retrofit:retrofit2-kotlinx-serialization-converter:1.0.0")
}
```

## Interface

```kotlin
interface ApiService {
    @GET("users")
    suspend fun getUsers(): List<User>
    
    @GET("users/{id}")
    suspend fun getUser(@Path("id") userId: Int): User
    
    @GET("search")
    suspend fun search(@Query("q") query: String): SearchResponse
    
    @POST("users")
    suspend fun createUser(@Body user: User): User
    
    @PUT("users/{id}")
    suspend fun updateUser(@Path("id") id: Int, @Body user: User): User
    
    @DELETE("users/{id}")
    suspend fun deleteUser(@Path("id") id: Int): Response<Unit>
}
```

## Retrofit Instance

```kotlin
val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .addConverterFactory(GsonConverterFactory.create())
    .build()

val apiService = retrofit.create(ApiService::class.java)
```

## Headers

```kotlin
interface ApiService {
    @Headers("Accept: application/json")
    @GET("users")
    suspend fun getUsers(): List<User>
    
    @GET("users")
    suspend fun getUsersWithToken(
        @Header("Authorization") token: String
    ): List<User>
}
```

## Recursos
- [Retrofit](https://square.github.io/retrofit/)
