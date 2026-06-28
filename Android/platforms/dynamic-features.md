# Dynamic Feature Delivery — Modularization Branch

## Overview

The `:dynamic` module demonstrates **Play Feature Delivery** — shipping parts of your app on-demand via Google Play instead of including everything in the initial install.

Android App Bundles (`.aab`) let Google Play split your app into a **base APK** + **dynamic feature APKs**. Users download the base APK first; dynamic features are delivered later.

---

## The `:dynamic` Module

This project's `:dynamic` is an **empty shell** — no Kotlin/Java source, just infrastructure:

```
dynamic/
├── build.gradle.kts
└── src/main/
    ├── AndroidManifest.xml
    └── res/values/strings.xml
```

It exists to demonstrate the **Gradle setup** for dynamic features, not to deliver actual functionality.

---

## Module Setup

### `dynamic/build.gradle.kts`

```kotlin
plugins {
    alias(libs.plugins.sample.dynamic)    // applies com.android.dynamic-feature
}

android {
    namespace = "com.google.samples.modularization.dynamic"
}

dependencies {
    implementation(project(":app:mobile"))   // dynamic feature MUST depend on base app
}
```

### `AndroidManifest.xml`

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:dist="http://schemas.android.com/apk/distribution"
    package="com.google.samples.modularization.dynamic">

    <dist:module
        dist:instant="false"
        dist:title="@string/title_dynamicfeature">
        <dist:delivery>
            <dist:install-time />
        </dist:delivery>
        <dist:fusing dist:include="true" />
    </dist:module>
</manifest>
```

### Manifest Attributes Explained

| Attribute | Value | Meaning |
|---|---|---|
| `dist:instant` | `false` | Not an instant app module |
| `dist:title` | string resource | Human-readable name in Play Console |
| `dist:install-time` | ✅ | Downloaded during app install (alongside base APK) |
| `dist:fusing include` | `true` | On pre-Android 5.0 devices, fuse this module into the base APK |

---

## Delivery Modes

| Mode | When downloaded | Use case |
|---|---|---|
| **install-time** | During app install | Features that should always be present (this project uses this) |
| **on-demand** | When app requests it | Optional features (AR filters, premium content) |
| **conditional** | Based on device conditions | Country-specific features, hardware-dependent features |

**This project uses install-time with fusing** — the simplest mode. The feature is always available, and the code is fused into the base APK on older devices.

---

## Base App Registration

`:app:mobile` declares `:dynamic` as a dynamic feature:

```kotlin
// app/mobile/build.gradle.kts
android {
    defaultConfig {
        applicationId = "com.google.samples.modularization"
    }
    dynamicFeatures += setOf(":dynamic")    // ← registers the dynamic module
}
```

This tells the Android Gradle plugin that `:dynamic` is a dynamic feature of this app. At build time, the AGP produces:
- A **base APK** from `:app:mobile`
- A **feature APK** from `:dynamic`
- An **Android App Bundle** (`.aab`) containing both

---

## Runtime: Checking Installed Modules

`MainActivity` logs which dynamic modules are installed:

```kotlin
val splitInstallManager = SplitInstallManagerFactory.create(this)
splitInstallManager.installedModules.addOnSuccessListener { moduleNames ->
    Log.e("MainActivity", "Installed dynamic module $moduleNames")
}
```

### Key APIs

| API | Purpose |
|---|---|
| `SplitInstallManagerFactory.create(context)` | Creates the split install manager |
| `splitInstallManager.installedModules` | `Set<String>` of installed module names |
| `splitInstallManager.installedModules.addOnSuccessListener {}` | Async check via `Task` API |

### For on-demand modules, you'd also use:

```kotlin
val request = SplitInstallRequest.newBuilder()
    .addModule("dynamic")
    .build()

splitInstallManager.startInstall(request)
    .addOnSuccessListener { /* module installed */ }
    .addOnFailureListener { /* handle error */ }
```

---

## Convention Plugin: `sample.dynamic`

```kotlin
class DynamicFeatureConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply {
                apply("com.android.dynamic-feature")
                apply("org.jetbrains.kotlin.android")
            }
            extensions.configure<DynamicFeatureExtension> {
                configureAndroid(this)    // compileSdk=33, minSdk=30, Java 17
            }
        }
    }
}
```

**Key differences from other convention plugins:**
- No Hilt plugin — dynamic features inherit Hilt from the base app
- No KAPT — Hilt annotation processing happens in the base app
- Applies `com.android.dynamic-feature` instead of `com.android.application` or `com.android.library`

---

## Fusing Explained

```xml
<dist:fusing dist:include="true" />
```

**Problem:** Dynamic feature modules require Android 5.0+ and the Play Store's split APK mechanism. On older devices or when side-loading, these features are missing.

**Solution:** Fusing merges the dynamic feature into the base APK for devices that can't handle split APKs. With `include="true"`, the feature is **always fused** — it's available on all devices regardless of Android version.

**Trade-off:** Fusing increases the base APK size (the feature code is included twice — once in the base, once as a separate APK).

---

## Dynamic Features vs Multiple App Modules

| Aspect | Dynamic Feature | Separate App Module (like `:app:wear`) |
|---|---|---|
| **Distribution** | Single app on Play Store, parts delivered on demand | Separate app listing |
| **Code sharing** | Shared with the base app (inherits dependencies) | Independent dependencies |
| **User install** | One install, features added later | Separate installs |
| **Use case** | Optional features in one app | Different form factors or app variants |

This project uses **both**: `:dynamic` for optional features within the mobile app, and `:app:wear` for a completely separate Wear OS app.

---

## When to Use Dynamic Features

### Use dynamic features when:
- You have features not all users need (premium, regional, device-specific)
- You want to reduce initial download size
- You're already using App Bundles (`.aab`)

### Don't use dynamic features when:
- The feature is core to the app experience
- You're distributing APKs directly (not via Play Store)
- The feature is tiny (overhead of split management isn't worth it)
