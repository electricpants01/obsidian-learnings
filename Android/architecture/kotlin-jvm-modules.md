# JVM-Only vs Android Library Modules — Modularization Branch

## The `:core:util` Pattern

`:core:util` is unique among this project's modules — it's the only one that uses **pure Kotlin/JVM** instead of the Android toolchain:

```kotlin
// core/util/build.gradle.kts
plugins {
    alias(libs.plugins.kotlin.jvm)    // ← org.jetbrains.kotlin.jvm
}
// That's the entire file. No android {}, no dependencies, no namespace.
```

Compare to an Android library module:

```kotlin
// core/ui/build.gradle.kts
plugins {
    alias(libs.plugins.sample.android.library)    // applies 4 plugins
    alias(libs.plugins.sample.compose)            // applies Compose + BOM
}
android { namespace = "com.google.samples.modularization.core.ui" }
dependencies {
    implementation(libs.androidx.compose.ui)
    implementation(libs.androidx.compose.material3)
}
```

---

## What `kotlin-jvm` Gives You

| Aspect | `kotlin-jvm` | `android-library` |
|---|---|---|
| **Compiles to** | JVM bytecode (`.class` → `.jar`) | Android bytecode (`.class` → `.dex`) |
| **Available APIs** | Kotlin stdlib + JVM stdlib | Kotlin stdlib + Android SDK |
| **Build time** | ~1-2 seconds | ~5-15 seconds (dexing, resource processing) |
| **Test framework** | JUnit on host JVM | JUnit on device/emulator (or Robolectric) |
| **Gradle plugin** | `kotlin-jvm` | `com.android.library` + `kotlin-android` |
| **Can use `android.*`** | ❌ No | ✅ Yes |
| **Can use `java.*`** | ✅ Full JVM | ✅ Subset (no AWT, Swing, etc.) |

---

## `:core:util` Source Code

```kotlin
package com.google.samples.modularization.util

import java.text.SimpleDateFormat
import java.util.Date
import java.util.Locale

fun timestampToReadableDate(
    timestamp: Long,
    format: String = "yyyy-MM-dd HH:mm:ss"
): String {
    val date = Date(timestamp)
    val simpleDateFormat = SimpleDateFormat(format, Locale.getDefault())
    return simpleDateFormat.format(date)
}
```

This function:
- Uses `java.text.SimpleDateFormat` and `java.util.Date` — available in both JVM and Android
- Does not touch `android.*` packages
- Is a **top-level Kotlin extension function** — no class wrapper needed

---

## Why Separate It Into Its Own Module?

### Reason 1: Fast Unit Tests

If `timestampToReadableDate()` lived in `:app:mobile` or `:feature:list`, testing it would require an Android emulator or Robolectric. In `:core:util` (kotlin-jvm), it tests on the host JVM:

```kotlin
// Could live in :core:util/src/test/ (not currently present, but easily added)
class TimestampTest {
    @Test
    fun `formats timestamp correctly`() {
        val result = timestampToReadableDate(0, "yyyy-MM-dd")
        assertEquals("1970-01-01", result)
    }
}
```

Runs in milliseconds on the JVM — no emulator startup.

### Reason 2: Reusable Without Android

`:feature:list` depends on `:core:util`:

```kotlin
// feature/list/build.gradle.kts
dependencies {
    implementation(project(":core:util"))
}
```

But `:feature:list` doesn't care that `:core:util` is JVM-only. It just calls the function. The Gradle dependency graph handles the rest — both JVM and Android modules can depend on a `kotlin-jvm` module.

### Reason 3: Clear Boundaries

Separating pure Kotlin from Android code enforces:
- No accidental `Context` parameter creep
- No `Handler(Looper.getMainLooper())` in utility code
- The function is forced to be platform-independent

---

## What Other Code Could Go in a `kotlin-jvm` Module?

| Type | Example |
|---|---|
| Date/time formatting | `timestampToReadableDate()` (this project) |
| String manipulation | Email validation, URL parsing |
| Math/algorithm code | Sorting, filtering, calculation logic |
| Data classes / domain models | `MyModel` (currently in `:data` with Android) |
| Repository interfaces | `MyModelRepository` interface (no Android needed) |

**Note:** This project puts `MyModel` and `MyModelRepository` in `:data` (an android-library). They could be extracted to a `:core:model` (kotlin-jvm) module if you wanted even faster test feedback on the domain layer.

---

## When to Use `kotlin-jvm` vs `android-library`

### Use `kotlin-jvm` when:
- The code never touches `android.*` packages
- You want fast JVM test feedback
- The code is pure domain logic or data transformation
- The module is consumed by non-Android targets (e.g., a backend sharing models)

### Use `android-library` when:
- Any code in the module touches `Context`, `View`, `Lifecycle`, etc.
- The module has resources (`res/`, `AndroidManifest.xml`)
- The module uses Android-specific dependencies (Room, WorkManager, Compose)
- Tests need an emulator or Robolectric

### Gray area — `:data` in this project:
```kotlin
dependencies {
    implementation(libs.kotlinx.coroutines.android)  // ← Android dependency
}
```

`DefaultMyModelRepository` uses `kotlinx-coroutines-android` (not `kotlinx-coroutines-core`), which adds the `Dispatchers.Main` that requires Android. If it used `kotlinx-coroutines-core`, it could be a `kotlin-jvm` module. This is a trade-off — Android scope for convenience vs pure JVM for speed.

---

## Module Type Summary

| Module | Plugin | Why |
|---|---|---|
| `:app:mobile` | `sample.android.application` | Produces an APK |
| `:app:wear` | `sample.android.application` | Produces an APK |
| `:feature:*` | `sample.android.library` | Uses Compose + Android ViewModels |
| `:core:ui` | `sample.android.library` | Uses Compose (needs Android) |
| `:core:testing` | `sample.android.library` | Uses Android test runner APIs |
| `:data` | `sample.android.library` | Uses `kotlinx-coroutines-android` |
| `:core:util` | **`kotlin-jvm`** | Pure Kotlin, no Android dependency |
| `:dynamic` | `sample.dynamic` | Dynamic feature module |
| `:test:navigation` | `sample.android.test` | Instrumentation test module |
