# Firebase Auth

## Setup

```swift
import FirebaseAuth

// Sign in
Auth.auth().signIn(withEmail: email, password: password) { result, error in
    if let error = error {
        print("Error: \(error)")
        return
    }
    print("Logged in!")
}
```

## Recursos
- [Firebase Auth](https://firebase.google.com/docs/auth)
