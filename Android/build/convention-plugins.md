# Gradle Convention Plugins — Modularization Branch

## What Problem Do They Solve?

Without convention plugins, every module's `build.gradle.kts` would have 30+ lines of repeated configuration:

```kotlin
// ❌ Without convention plugins — every module repeats this
android {
    compileSdk = 33
    defaultConfig {
        minSdk = 30
        testInstrumentationRunner = "...HiltTestRunner"
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    buildFeatures { buildConfig = false }
}
dependencies {
    implementation("com.google.dagger:hilt-android:2.45")
    kapt("com.google.dagger:hilt-compiler:2.45")
    androidTestImplementation("com.google.dagger:hilt-android-testing:2.45")
    kaptAndroidTest("com.google.dagger:hilt-android-compiler:2.45")
}
```

With convention plugins, each module declares only what's unique:

```kotlin
// ✅ With convention plugins — one line of plugins, 3 lines of deps
plugins {
    alias(libs.plugins.sample.android.library)
    alias(libs.plugins.sample.compose)
}
android { namespace = "com.google.samples.modularization.feature.list" }
dependencies {
    implementation(project(":core:ui"))
    implementation(project(":data"))
}
```

---

## Architecture

```
project-root/
├── build-logic/                          ← included build
│   ├── settings.gradle.kts               ← re-shares version catalog from parent
│   └── convention/
│       ├── build.gradle.kts              ← registers 5 plugins
│       └── src/main/kotlin/
│           ├── Android.kt                ← shared helper: configureAndroid()
│           ├── Compose.kt                ← shared helper: configureCompose()
│           ├── AndroidApplicationConventionPlugin.kt
│           ├── AndroidLibraryConventionPlugin.kt
│           ├── AndroidTestConventionPlugin.kt
│           ├── ComposeConventionPlugin.kt
│           └── DynamicFeatureConventionPlugin.kt
└── settings.gradle.kts
    pluginManagement { includeBuild("build-logic") }  ← makes plugins available
```

**`includeBuild("build-logic")`** makes `build-logic` an **included build** — its Gradle plugins are available to all modules in the main project without publishing to a repository.

---

## Plugin Registration

`build-logic/convention/build.gradle.kts` maps plugin IDs to implementation classes:

```kotlin
plugins { `kotlin-dsl` }

gradlePlugin {
    plugins {
        register("sampleAndroidApplication") {
            id = "sample.android.application"
            implementationClass = "AndroidApplicationConventionPlugin"
        }
        register("sampleAndroidLibrary") {
            id = "sample.android.library"
            implementationClass = "AndroidLibraryConventionPlugin"
        }
        register("sampleAndroidTest") {
            id = "sample.android.test"
            implementationClass = "AndroidTestConventionPlugin"
        }
        register("sampleCompose") {
            id = "sample.compose"
            implementationClass = "ComposeConventionPlugin"
        }
        register("sampleDynamic") {
            id = "sample.dynamic"
            implementationClass = "DynamicFeatureConventionPlugin"
        }
    }
}
```

`gradle/libs.versions.toml` makes them referenceable via `alias()`:

```toml
[plugins]
sample-android-application = { id = "sample.android.application", version = "unspecified" }
sample-android-library = { id = "sample.android.library", version = "unspecified" }
sample-android-test = { id = "sample.android.test", version = "unspecified" }
sample-compose = { id = "sample.compose", version = "unspecified" }
sample-dynamic = { id = "sample.dynamic", version = "unspecified" }
```

---

## The 5 Convention Plugins

### 1. `sample.android.application`

**Applied by:** `:app:mobile`, `:app:wear`

```kotlin
class AndroidApplicationConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply {
                apply("com.android.application")
                apply("dagger.hilt.android.plugin")
                apply("org.jetbrains.kotlin.android")
                apply("org.jetbrains.kotlin.kapt")
            }
            extensions.configure<BaseAppModuleExtension> {
                configureAndroid(this)       // compileSdk=33, minSdk=30, Java 17, HiltTestRunner
            }
            dependencies {
                add("implementation", libs.hilt.android)
                add("kapt", libs.hilt.compiler)
                add("androidTestImplementation", libs.hilt.android.testing)
                add("kaptAndroidTest", libs.hilt.android.compiler)
            }
            kaptExtension { correctErrorTypes = true }
        }
    }
}
```

**Provides:**
- `com.android.application` plugin
- Hilt plugin + dependencies + KAPT
- Kotlin Android + KAPT plugins
- `configureAndroid()` → compileSdk, minSdk, Java 17, test runner

### 2. `sample.android.library`

**Applied by:** `:core:ui`, `:core:testing`, `:data`, `:feature:list`, `:feature:details`, `:feature:wear:home`

Same as application but applies `com.android.library` instead of `com.android.application`. Uses `LibraryExtension` for `configureAndroid()`.

### 3. `sample.android.test`

**Applied by:** `:test:navigation`

```kotlin
class AndroidTestConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply {
                apply("com.android.test")
                apply("org.jetbrains.kotlin.android")
                apply("org.jetbrains.kotlin.kapt")
            }
            extensions.configure<TestExtension> { configureAndroid(this) }
            dependencies {
                add("implementation", libs.test.core)
                add("implementation", libs.compose.ui.test.junit4)
                add("implementation", libs.hilt.android)
                add("implementation", libs.hilt.android.testing)
                add("kapt", libs.hilt.compiler)
            }
            kaptExtension { correctErrorTypes = true }
        }
    }
}
```

**Key difference:** Applies `com.android.test` (not library), adds test-specific deps (test-core, compose-ui-test-junit4).

### 4. `sample.compose`

**Applied by:** `:app:mobile`, `:app:wear`, `:core:ui`, `:feature:list`, `:feature:details`, `:feature:wear:home`

```kotlin
class ComposeConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            val extension = extensions.getByType<BaseExtension>()
            configureCompose(extension)
        }
    }
}
```

Delegates to `Compose.kt`:

```kotlin
fun Project.configureCompose(commonExtension: BaseExtension) {
    commonExtension.apply {
        buildFeatures { compose = true }
        composeOptions {
            kotlinCompilerExtensionVersion = libs.findVersion("androidxComposeCompiler").get()
        }
        dependencies {
            add("implementation", platform(libs.androidx.compose.bom))
            add("androidTestImplementation", libs.compose.ui.test.junit4)
            add("androidTestImplementation", project(":core:testing"))
        }
    }
}
```

**Provides:**
- `buildFeatures { compose = true }`
- Compose compiler extension version from version catalog
- Compose BOM platform dependency (manages Compose artifact versions)
- Compose UI test + `:core:testing` as androidTestImplementation

### 5. `sample.dynamic`

**Applied by:** `:dynamic`

```kotlin
class DynamicFeatureConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply {
                apply("com.android.dynamic-feature")
                apply("org.jetbrains.kotlin.android")
            }
            extensions.configure<DynamicFeatureExtension> {
                configureAndroid(this)
            }
        }
    }
}
```

**Key difference:** No Hilt plugin — dynamic features inherit Hilt from the base app. No KAPT.

---

## Shared Helper Functions

### `Android.kt`

```kotlin
fun configureAndroid(commonExtension: CommonExtension<*, *, *, *>) {
    commonExtension.apply {
        compileSdk = 33
        defaultConfig {
            minSdk = 30
            testInstrumentationRunner = "com.google.samples.modularization.testing.HiltTestRunner"
        }
        compileOptions {
            sourceCompatibility = JavaVersion.VERSION_17
            targetCompatibility = JavaVersion.VERSION_17
        }
        buildFeatures.buildConfig = false
    }
}
```

Used by all 5 convention plugins. The `CommonExtension<*, *, *, *>` type is the base type for Application, Library, Test, and DynamicFeature extensions — one function works for all.

### `Compose.kt`

```kotlin
fun Project.configureCompose(commonExtension: BaseExtension) {
    commonExtension.apply {
        buildFeatures { compose = true }
        composeOptions {
            kotlinCompilerExtensionVersion =
                libs.findVersion("androidxComposeCompiler").get().toString()
        }
        dependencies {
            add("implementation", platform(libs.findLibrary("androidx.compose.bom").get()))
            add("androidTestImplementation", libs.findLibrary("androidx.compose.ui.test.junit4").get())
            add("androidTestImplementation", project(":core:testing"))
        }
    }
}
```

---

## What the Version Catalog Looks Like

```toml
[versions]
androidGradlePlugin = "7.4.2"
hilt = "2.45"
kotlin = "1.8.10"
androidxComposeCompiler = "1.4.3"
androidxComposeBom = "2023.03.00"

[libraries]
hilt-android = { module = "com.google.dagger:hilt-android", version.ref = "hilt" }
hilt-compiler = { module = "com.google.dagger:hilt-compiler", version.ref = "hilt" }
# ... 30+ entries

[plugins]
sample-android-library = { id = "sample.android.library", version = "unspecified" }
sample-compose = { id = "sample.compose", version = "unspecified" }
```

---

## Plugin Composition: Which Plugins Per Module

| Module | sample.android.application | sample.android.library | sample.compose | sample.android.test | sample.dynamic | kotlin.jvm |
|---|---|---|---|---|---|---|
| `:app:mobile` | ✅ | | ✅ | | | |
| `:app:wear` | ✅ | | ✅ | | | |
| `:core:ui` | | ✅ | ✅ | | | |
| `:core:util` | | | | | | ✅ |
| `:core:testing` | | ✅ | | | | |
| `:data` | | ✅ | | | | |
| `:feature:list` | | ✅ | ✅ | | | |
| `:feature:details` | | ✅ | ✅ | | | |
| `:feature:wear:home` | | ✅ | ✅ | | | |
| `:dynamic` | | | | | ✅ | |
| `:test:navigation` | | | | ✅ | | |

---

## Why Convention Plugins Over Other Approaches?

| Approach | Problem |
|---|---|
| **Copy-paste `build.gradle.kts`** | Same config in 12 files → version drift, hard to update |
| **`apply from: "shared.gradle"` (script plugins)** | No type safety, no IDE autocomplete, fragile |
| **`subprojects {}` in root `build.gradle.kts`** | Forces same config on all modules, no opt-in/opt-out |
| **Convention plugins via `build-logic`** | Type-safe, IDE-supported, explicit opt-in, single source of truth |

Convention plugins give you: **type safety, IDE support, explicit opt-in per module, and a single place to update shared config.**
