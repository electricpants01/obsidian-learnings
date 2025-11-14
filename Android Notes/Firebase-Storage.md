# Firebase Storage

## Setup

```kotlin
dependencies {
    implementation("com.google.firebase:firebase-storage-ktx")
}
```

## Upload

```kotlin
val storage = Firebase.storage
val storageRef = storage.reference

val fileRef = storageRef.child("images/${UUID.randomUUID()}.jpg")
fileRef.putFile(fileUri)
    .addOnSuccessListener {
        fileRef.downloadUrl.addOnSuccessListener { uri ->
            val downloadUrl = uri.toString()
        }
    }
```

## Download

```kotlin
val fileRef = storageRef.child("images/photo.jpg")
fileRef.getFile(localFile)
    .addOnSuccessListener {
        // File downloaded
    }
```

## Recursos
- [Firebase Storage](https://firebase.google.com/docs/storage/android/start)
