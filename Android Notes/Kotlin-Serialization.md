# Kotlin Serialization

## Setup

```kotlin
// build.gradle (project)
plugins {
    id("org.jetbrains.kotlin.plugin.serialization") version "1.9.0"
}

// build.gradle (module)
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.0")
}
```

## Serializable Classes

```kotlin
@Serializable
data class User(
    val id: Int,
    val name: String,
    val email: String? = null
)

// Serializar
val user = User(1, "John", "john@example.com")
val json = Json.encodeToString(user)

// Deserializar
val userData = Json.decodeFromString<User>(json)
```

## Custom Names

```kotlin
@Serializable
data class User(
    @SerialName("user_id")
    val id: Int,
    @SerialName("user_name")
    val name: String
)
```

## Transient

```kotlin
@Serializable
data class User(
    val id: Int,
    val name: String,
    @Transient
    val password: String = "" // No se serializa
)
```

## Recursos
- [Kotlin Serialization](https://kotlinlang.org/docs/serialization.html)
