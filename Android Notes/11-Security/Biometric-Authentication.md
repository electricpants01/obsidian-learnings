# Biometric Authentication

## Setup

```kotlin
dependencies {
    implementation("androidx.biometric:biometric:1.1.0")
}
```

## Uso

```kotlin
val executor = ContextCompat.getMainExecutor(context)
val biometricPrompt = BiometricPrompt(activity, executor,
    object : BiometricPrompt.AuthenticationCallback() {
        override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
            // Success
        }
        
        override fun onAuthenticationError(errorCode: Int, errString: CharSequence) {
            // Error
        }
    }
)

val promptInfo = BiometricPrompt.PromptInfo.Builder()
    .setTitle("Biometric Authentication")
    .setSubtitle("Authenticate to continue")
    .setNegativeButtonText("Cancel")
    .build()

biometricPrompt.authenticate(promptInfo)
```

## Recursos
- [Biometric](https://developer.android.com/training/sign-in/biometric-auth)
