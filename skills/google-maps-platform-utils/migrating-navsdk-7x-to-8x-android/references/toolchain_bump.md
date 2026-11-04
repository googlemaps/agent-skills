# AGP 9 / Gradle 9 Toolchain Bump (NavSDK 8.0)

NavSDK 8.0 AAR metadata requires **Android Gradle Plugin 9.0.1+** and **Gradle
9.1.0+**. Use **JDK 17 minimum (JDK 21 recommended)**. Do not move to JDK 25 as
part of this migration.

Bump the toolchain first and get a green build **before** touching NavSDK APIs.
That way any build failure can be traced to either the toolchain or the API
changes, but not both.

For anything not listed here, follow the official
[AGP 9.0 release notes](https://developer.android.com/build/releases/agp-9-0-0-release-notes)
instead of restating them.

## Version targets

*   `com.android.application` / `library`: `9.0.1`
*   Gradle wrapper (`distributionUrl`): `gradle-9.1.0-bin.zip` for AGP 9.0.1;
    newer AGP 9.x needs newer Gradle (AGP 9.2.0 needs 9.4.1)
*   JDK (Gradle daemon): 17 (min), 21 (preferred)
*   `compileSdk` / `targetSdk`: `37`
*   `com.google.gms:google-services`: `4.5.0` (if used)
*   `org.chromium.net:cronet-fallback`: `143.7445.0` (if pinned)

## Symptom to fix

Each entry is the build error or warning, followed by the fix.

*   `package="..."` attribute in `AndroidManifest.xml` is rejected: Remove
    `package=` from the manifest and set `android { namespace = "..." }` in the
    module's build file.
*   `Plugin 'org.jetbrains.kotlin.android' ... already applied` / Kotlin plugin
    conflict: AGP 9 has built-in Kotlin. Remove `kotlin-android` from
    submodules. If the root build declares it, keep it as `apply false` only.
*   `kotlinOptions` is unresolved: Replace it with `kotlin { compilerOptions {
    jvmTarget.set(org.jetbrains.kotlin.gradle.dsl.JvmTarget.JVM_11) } }` (or
    17).
*   `dexOptions` is unresolved: Delete the `dexOptions { ... }` block. It has no
    replacement.
*   Old support-library classes are missing at compile or runtime:
    `android.enableJetifier` now defaults to `false`. Migrate the offending
    dependency to AndroidX. Only as a temporary measure, set
    `android.enableJetifier=true` in `gradle.properties`.
*   R8 fails on `-dontobfuscate` / `-dontoptimize` / `-dontshrink` coming from a
    consumer/library rule file: R8 now treats these global options in consumer
    rules as errors. Remove them from the app's own rules, and upgrade or
    exclude the library that ships them.
*   `getDefaultProguardFile('proguard-android.txt')` is rejected: Switch to
    `getDefaultProguardFile("proguard-android-optimize.txt")`.
*   `google-services` plugin is incompatible with AGP 9: Bump
    `com.google.gms:google-services` to `4.5.0`.
*   Duplicate / mismatched Cronet classes: If the app pins `cronet-fallback`,
    align it to `143.7445.0`.
*   `Unsupported class file major version 69`: The Gradle daemon is running on
    JDK 25. Point `org.gradle.java.home` / Android Studio's Gradle JDK at JDK 17
    or 21.
*   R8 task (`minify<Variant>WithR8`) fails: proguard file not found: AGP 9
    fails on a `proguardFiles` entry that doesn't exist (AGP 8 ignored it).
    Create the file or remove the reference.
*   `Minimum supported Gradle version is X` (e.g. AGP 9.2.0 on Gradle 9.1.0):
    Bump `distributionUrl` in `gradle/wrapper/gradle-wrapper.properties` to the
    version in the message.
*   `failed to find target android-37` on the first AGP 9 build: AGP
    auto-installed the platform mid-build. Re-run the build.
*   `kotlin { compilerOptions { ... } }` unresolved inside `android {}`: Put the
    `kotlin {}` block at the **top level** of the module build file, not inside
    `android {}`.
*   Deprecation warnings: `resConfigs`, `buildDir`, Groovy `prop value`
    assignment: Optional cleanup, not build-breaking: `androidResources {
    localeFilters += ["en"] }`, `layout.buildDirectory`, `prop = value`. Gradle
    10 will remove the old forms.
*   "tested up to compile SDK version 36.1" warning: Expected with compileSdk 37
    on AGP 9.0.1. Optionally set `android.suppressUnsupportedCompileSdk=37.0` in
    `gradle.properties`.

## Verify

```shell
./gradlew --version            # Gradle 9.1.0, JVM 17 or 21
./gradlew :app:assembleDebug :app:assembleRelease
```

Both variants must build. **Release** is where R8 consumer-rule errors appear.
