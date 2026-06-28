# Build Optimization — Modularization Branch

## `gradle.properties` — The Full File

```properties
# JVM heap for the Gradle daemon
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8

# AndroidX
android.useAndroidX=true

# Kotlin code style
kotlin.code.style=official

# Non-transitive R classes (per-module R files)
android.nonTransitiveRClass=true

# Build speed optimizations
org.gradle.caching=true
org.gradle.parallel=true
org.gradle.unsafe.configuration-cache=true
```

---

## Optimization 1: Build Caching (`org.gradle.caching=true`)

**What it does:** Gradle stores task outputs (compiled classes, processed resources) in a local cache. On the next build, if inputs haven't changed, Gradle reuses the cached output instead of re-running the task.

**Impact on this project:** Without caching, changing one file in `:feature:list` would recompile every module that depends on it. With caching, only `:feature:list` and `:app:mobile` recompile — all intermediate modules (`:core:ui`, `:data`) use cached outputs.

**How to use effectively:**
- Cache is local by default (in `~/.gradle/caches/build-cache-1/`)
- Remote cache (`--build-cache` + remote push) for CI → local reuse
- Inputs must be deterministic — avoid `new Date()` or random values in build scripts

---

## Optimization 2: Parallel Execution (`org.gradle.parallel=true`)

**What it does:** Gradle executes independent tasks concurrently. Modules that don't depend on each other build simultaneously.

**Impact on this project:**

```
Without parallel:          With parallel:
:core:ui ──► 5s            :core:ui    ──► 5s
:data ──────► 3s            :data       ──► 3s       (runs simultaneously)
:feature:list ──► 4s        :feature:list ──► 4s      (runs simultaneously)
:app:mobile ──► 6s          :app:mobile ──► 6s
─────────────────           ─────────────────
Total: 18s                  Total: ~10s
```

**Requirements for parallel builds:**
- Projects must be **decoupled** — modules don't rely on sibling project directories
- No shared mutable state in build scripts
- This project qualifies because each module is self-contained

---

## Optimization 3: Configuration Cache (`org.gradle.unsafe.configuration-cache=true`)

**What it does:** Gradle caches the result of the **configuration phase** (the phase where it evaluates `build.gradle.kts` files and constructs the task graph). On subsequent builds, it skips configuration entirely and goes straight to execution.

**Why it's marked "unsafe":** As of Gradle 7.x, the configuration cache is experimental. It works well with convention plugins and version catalogs (this project's setup) but can fail with plugins that use project-level mutable state.

**Impact:** In a 12-module project, the configuration phase might take 3-8 seconds evaluating all `build.gradle.kts` files. Configuration cache eliminates this after the first run.

**What breaks configuration cache:**
- `beforeEvaluate` / `afterEvaluate` hooks
- Accessing `project.properties` at configuration time
- Calling `System.getenv()` or reading files during configuration
- This project **avoids all of these** — it uses convention plugins and version catalogs exclusively

---

## Optimization 4: Non-Transitive R Classes (`android.nonTransitiveRClass=true`)

**What it does:** Each library module's `R` class contains **only** the resources declared in that module, not resources from its transitive dependencies.

**Without the flag:**
```
:core:ui's R.class contains:
  - R.string.core_ui_resource
  - R.string.feature_list_resource    ← inherited from :feature:list? No!
  Wait — modules can't depend on each other, but the point stands:
  R classes merge transitively, bloating each module's R file.
```

**With the flag (this project):**
```
:core:ui's R.class → only core:ui resources
:feature:list's R.class → only feature:list resources
:app:mobile's R.class → only app:mobile resources
```

**Benefits:**
- Smaller `R` classes → faster compilation
- Non-constant resource IDs → smaller incremental changes to R
- Clearer ownership — each module owns only its own resources

**Note:** This requires `android.namespace` in every module's `build.gradle.kts` (not `package` in `AndroidManifest.xml`), which this project does.

---

## Optimization 5: `buildFeatures.buildConfig = false`

Set in `Android.kt` (applied to all modules by convention plugins):

```kotlin
buildFeatures.buildConfig = false
```

**What it does:** Disables `BuildConfig` class generation. `BuildConfig` contains `DEBUG`, `APPLICATION_ID`, `VERSION_NAME`, etc. If no code reads `BuildConfig`, generating it is wasted time.

**This project:** No code references `BuildConfig.DEBUG` or any other `BuildConfig` field, so this saves build time and class count.

**If you need `BuildConfig`:** Set `buildConfig = true` only in the module that needs it (override in that module's `build.gradle.kts`).

---

## Optimization 6: `FAIL_ON_PROJECT_REPOS`

In root `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
```

**What it does:** Prevents individual modules from declaring their own `repositories {}` block. All dependency resolution goes through the centralized repository list.

**Why it matters for builds:**
- Prevents accidental repository divergence between modules
- Faster resolution — Gradle doesn't need to merge per-module repos
- Security — no module can pull from an untrusted repository

---

## Optimization 7: `org.gradle.configureondemand=true`

Set in `build-logic/gradle.properties`:

```properties
org.gradle.configureondemand=true
```

**What it does:** Gradle only configures modules needed for the current task, not every module in the project.

**Example:**
```bash
./gradlew :data:test
# Without configureondemand: configures all 12 modules
# With configureondemand: configures only :data and its dependencies
```

**Caveat:** Only works well with decoupled projects. This project qualifies.

---

## Optimization 8: Gradle Daemon (`-Xmx2048m`)

```properties
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
```

- **2 GB heap** for the Gradle daemon — prevents GC pauses during large builds
- **UTF-8 encoding** ensures consistent string handling across platforms

---

## Optimization 9: Kotlin-DSL for Convention Plugins

```kotlin
// build-logic/convention/build.gradle.kts
plugins { `kotlin-dsl` }
```

Kotlin DSL provides:
- **Type-safe build scripts** — IDE autocomplete, compile-time error detection
- **Shared code** — `Android.kt` and `Compose.kt` are actual Kotlin files, not scripts
- **Plugin registration** — `gradlePlugin {}` block with typed `implementationClass`

---

## Build Time Impact Summary

| Optimization | Phase | Impact |
|---|---|---|
| `org.gradle.caching=true` | Execution | Skips re-running tasks with unchanged inputs |
| `org.gradle.parallel=true` | Execution | Builds independent modules simultaneously |
| `org.gradle.configuration-cache=true` | Configuration | Skips re-evaluating `build.gradle.kts` files |
| `android.nonTransitiveRClass=true` | Compilation | Smaller R classes → faster compilation |
| `buildFeatures.buildConfig = false` | Compilation | One fewer generated class per module |
| `FAIL_ON_PROJECT_REPOS` | Resolution | Single repository list, no merge overhead |
| `configureondemand=true` | Configuration | Only configures needed modules |
| 2 GB daemon heap | All phases | Fewer GC pauses |
