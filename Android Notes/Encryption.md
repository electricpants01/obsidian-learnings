# Encryption

## Keystore

```kotlin
val keyGenerator = KeyGenerator.getInstance(
    KeyProperties.KEY_ALGORITHM_AES,
    "AndroidKeyStore"
)

val keyGenParameterSpec = KeyGenParameterSpec.Builder(
    "MyKeyAlias",
    KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT
).setBlockModes(KeyProperties.BLOCK_MODE_GCM)
 .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
 .build()

keyGenerator.init(keyGenParameterSpec)
val secretKey = keyGenerator.generateKey()
```

## Cipher

```kotlin
val cipher = Cipher.getInstance("AES/GCM/NoPadding")
cipher.init(Cipher.ENCRYPT_MODE, secretKey)
val encryptedData = cipher.doFinal(plainText.toByteArray())
```

## Recursos
- [Encryption](https://developer.android.com/topic/security/data)
