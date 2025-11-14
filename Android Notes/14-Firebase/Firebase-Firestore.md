# Firebase Firestore

## Setup

```kotlin
dependencies {
    implementation("com.google.firebase:firebase-firestore-ktx")
}
```

## CRUD

```kotlin
val db = Firebase.firestore

// Create
db.collection("users").add(user)

// Read
db.collection("users").document(userId).get()
    .addOnSuccessListener { document ->
        val user = document.toObject<User>()
    }

// Update
db.collection("users").document(userId).update("name", newName)

// Delete
db.collection("users").document(userId).delete()

// Listen
db.collection("users").addSnapshotListener { snapshot, error ->
    snapshot?.documents?.forEach { doc ->
        val user = doc.toObject<User>()
    }
}
```

## Recursos
- [Firestore](https://firebase.google.com/docs/firestore/quickstart)
