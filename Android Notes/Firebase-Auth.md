# Firebase Authentication

## Setup

```kotlin
dependencies {
    implementation(platform("com.google.firebase:firebase-bom:32.7.0"))
    implementation("com.google.firebase:firebase-auth-ktx")
}
```

## Email/Password

```kotlin
val auth = Firebase.auth

// Sign up
auth.createUserWithEmailAndPassword(email, password)
    .addOnCompleteListener { task ->
        if (task.isSuccessful) {
            val user = auth.currentUser
        }
    }

// Sign in
auth.signInWithEmailAndPassword(email, password)
    .addOnCompleteListener { task ->
        if (task.isSuccessful) {
            val user = auth.currentUser
        }
    }

// Sign out
auth.signOut()
```

## Google Sign In

```kotlin
val googleSignInClient = GoogleSignIn.getClient(context, gso)
val signInIntent = googleSignInClient.signInIntent
launcher.launch(signInIntent)

// Handle result
val credential = GoogleAuthProvider.getCredential(account.idToken, null)
auth.signInWithCredential(credential)
```

## Recursos
- [Firebase Auth](https://firebase.google.com/docs/auth/android/start)
