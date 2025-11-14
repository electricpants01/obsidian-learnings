# Instant Apps

## Setup

```kotlin
// build.gradle.kts
android {
    buildFeatures {
        instantApps = true
    }
}
```

## Instant Feature Module

```kotlin
plugins {
    id("com.android.instantapp")
}

dependencies {
    implementation(project(":base"))
    implementation(project(":feature"))
}
```

## URL Mapping

```xml
<activity>
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="https"
              android:host="example.com" />
    </intent-filter>
</activity>
```

## Recursos
- [Instant Apps](https://developer.android.com/topic/google-play-instant)
