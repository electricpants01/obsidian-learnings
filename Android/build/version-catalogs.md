# Version Catalogs (`libs.versions.toml`) — Modularization Branch

## What Is a Version Catalog?

A single TOML file that declares all dependency coordinates used across the entire project. Every module references dependencies through the catalog instead of hardcoding Maven coordinates.

**Location:** `gradle/libs.versions.toml`

---

## The Full Catalog

```toml
[versions]
androidGradlePlugin = "7.4.2"
androidxActivity = "1.6.1"
androidxComposeBom = "2023.03.00"
androidxComposeCompiler = "1.4.3"
androidxCore = "1.9.0"
androidxHilt = "1.0.0"
androidxLifecycle = "2.6.0"
androidxNavigation = "2.5.3"
androidxRoom = "2.5.0"
androidxTestCore = "1.5.0"
androidxTestExt = "1.1.5"
androidxTestRunner = "1.5.2"
compose = "1.2.1"
composeWear = "1.0.0"
coroutines = "1.6.4"
hilt = "2.45"
junit = "4.13.2"
kotlin = "1.8.10"

[libraries]
androidx-core-ktx = { module = "androidx.core:core-ktx", version.ref = "androidxCore" }
androidx-compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "androidxComposeBom" }
androidx-compose-material3 = { group = "androidx.compose.material3", name = "material3" }
hilt-android = { module = "com.google.dagger:hilt-android", version.ref = "hilt" }
hilt-compiler = { module = "com.google.dagger:hilt-compiler", version.ref = "hilt" }
junit = { module = "junit:junit", version.ref = "junit" }
kotlinx-coroutines-android = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-android", version.ref = "coroutines" }
kotlinx-coroutines-test = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-test", version.ref = "coroutines" }
# ... 30+ total entries

[plugins]
android-application = { id = "com.android.application", version.ref = "androidGradlePlugin" }
android-library = { id = "com.android.library", version.ref = "androidGradlePlugin" }
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
sample-android-application = { id = "sample.android.application", version = "unspecified" }
sample-android-library = { id = "sample.android.library", version = "unspecified" }
sample-compose = { id = "sample.compose", version = "unspecified" }
# ... 11 total entries
```

---

## Anatomy of a Catalog Entry

### Version Declaration
```toml
[versions]
hilt = "2.45"
```
Declares a named version that can be referenced by multiple libraries.

### Library Declaration (two syntaxes)

**Maven coordinate shorthand (preferred):**
```toml
hilt-android = { module = "com.google.dagger:hilt-android", version.ref = "hilt" }
```

**Group + name (for BOM-only artifacts):**
```toml
androidx-compose-material3 = { group = "androidx.compose.material3", name = "material3" }
# No version — the Compose BOM manages it
```

### Plugin Declaration
```toml
[plugins]
sample-android-library = { id = "sample.android.library", version = "unspecified" }
# "unspecified" means the version comes from the included build (build-logic)
```

---

## How Modules Use the Catalog

### In `build.gradle.kts` — Dependencies

```kotlin
// Before: hardcoded coordinates
dependencies {
    implementation("com.google.dagger:hilt-android:2.45")
    kapt("com.google.dagger:hilt-compiler:2.45")
    testImplementation("junit:junit:4.13.2")
}

// After: catalog references
dependencies {
    implementation(libs.hilt.android)
    kapt(libs.hilt.compiler)
    testImplementation(libs.junit)
}
```

Dash-separated catalog keys become dot-separated accessors: `hilt-android` → `libs.hilt.android`.

### In `build.gradle.kts` — Plugins

```kotlin
// Before: hardcoded plugin ID + version
plugins {
    id("com.android.library") version "7.4.2"
    id("org.jetbrains.kotlin.android") version "1.8.10"
}

// After: catalog alias
plugins {
    alias(libs.plugins.sample.android.library)
    // Kotlin + Hilt + KAPT are applied by the convention plugin
}
```

### In Convention Plugins (Programmatic Access)

Convention plugins use the catalog via `extensions.getByType<VersionCatalogsExtension>()`:

```kotlin
val libs = extensions.getByType<VersionCatalogsExtension>().named("libs")

dependencies {
    add("implementation", libs.findLibrary("hilt.android").get())
    add("kapt", libs.findLibrary("hilt.compiler").get())
}
```

The `build-logic` included build shares the parent's version catalog:

```kotlin
// build-logic/settings.gradle.kts
dependencyResolutionManagement {
    versionCatalogs {
        create("libs") {
            from(files("../gradle/libs.versions.toml"))  // ← re-shares parent catalog
        }
    }
}
```

---

## Compose BOM: A Special Case

The Compose BOM is applied as a **platform dependency**:

```kotlin
dependencies {
    val composeBom = platform(libs.androidx.compose.bom)
    implementation(composeBom)
}
```

**How BOMs work:** The BOM declares versions for all Compose artifacts. When a module depends on `androidx.compose.material3` without specifying a version, the BOM provides it. This ensures all Compose artifacts in the project use the same versions.

**Catalog entry:**
```toml
androidx-compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "androidxComposeBom" }
androidx-compose-material3 = { group = "androidx.compose.material3", name = "material3" }
# No version on material3 — the BOM manages it
```

---

## Why Version Catalogs Instead of Other Approaches?

| Approach | Problems |
|---|---|
| **Hardcoded versions** | `"2.45"` scattered across 12 build files — version drift, hard to audit |
| **`ext { }` in root `build.gradle`** | No type safety, no IDE autocomplete, stringly-typed |
| **Gradle `platform()` for everything** | Works for libraries but not for plugins |
| **Version catalogs** | Type-safe, IDE-supported, works for libraries + plugins, TOML is human-readable |

---

## What You Learn From This Project's Catalog

| Concept | Example |
|---|---|
| **Version refs** | `version.ref = "hilt"` — reuse the same version across libraries |
| **BOM-managed artifacts** | Compose artifacts have no version — the BOM provides it |
| **`unspecified` plugin versions** | Convention plugins resolve from the included build |
| **Catalog sharing** | `build-logic/settings.gradle.kts` re-shares via `from(files(...))` |
| **Dot-separated accessors** | `hilt-android` → `libs.hilt.android` |
| **Group/name vs module syntax** | `{ group = "...", name = "..." }` for BOM-only artifacts |
