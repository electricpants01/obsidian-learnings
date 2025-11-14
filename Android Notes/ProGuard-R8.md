# ProGuard & R8

## ProGuard Rules

```proguard
# Keep classes
-keep class com.example.myapp.** { *; }

# Keep interfaces
-keep interface com.example.myapp.** { *; }

# Keep data classes for serialization
-keep class com.example.myapp.data.** { *; }

# Retrofit
-keepattributes Signature, InnerClasses, EnclosingMethod
-keepattributes RuntimeVisibleAnnotations, RuntimeVisibleParameterAnnotations

# Kotlin
-keep class kotlin.Metadata { *; }
```

## R8

```kotlin
android {
    buildTypes {
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```

## Recursos
- [R8](https://developer.android.com/studio/build/shrink-code)
