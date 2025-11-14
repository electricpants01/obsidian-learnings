# Ktor Client

## Setup

```kotlin
dependencies {
    implementation("io.ktor:ktor-client-android:2.3.7")
    implementation("io.ktor:ktor-client-content-negotiation:2.3.7")
    implementation("io.ktor:ktor-serialization-kotlinx-json:2.3.7")
}
```

## Cliente

```kotlin
val client = HttpClient(Android) {
    install(ContentNegotiation) {
        json(Json {
            ignoreUnknownKeys = true
        })
    }
    install(Logging) {
        level = LogLevel.ALL
    }
}

suspend fun getUsers(): List<User> {
    return client.get("https://api.example.com/users").body()
}

suspend fun createUser(user: User): User {
    return client.post("https://api.example.com/users") {
        contentType(ContentType.Application.Json)
        setBody(user)
    }.body()
}
```

## Recursos
- [Ktor Client](https://ktor.io/docs/getting-started-ktor-client.html)
