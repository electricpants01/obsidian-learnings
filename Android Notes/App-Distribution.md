# App Distribution

## Google Play

```kotlin
// Generar AAB
./gradlew bundleRelease

// Generar APK
./gradlew assembleRelease
```

## Signing

```kotlin
android {
    signingConfigs {
        create("release") {
            storeFile = file("keystore.jks")
            storePassword = "password"
            keyAlias = "key"
            keyPassword = "password"
        }
    }
    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
        }
    }
}
```

## App Bundle

```kotlin
// Beneficios: menor tamaño de descarga, APKs optimizados por dispositivo
```

## Recursos
- [Distribution](https://developer.android.com/studio/publish)
