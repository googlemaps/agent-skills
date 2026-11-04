---
name: migrating-navsdk-7x-to-8x-android
description: >-
  Upgrades an Android app (Java or Kotlin) that uses the Navigation SDK for
  Android (com.google.android.libraries.navigation) from 7.x to 8.0. Covers the
  AGP 9.0.1 / Gradle 9.1.0 toolchain floor, targetSdk 37 and the Play Location
  Purpose Declaration, removed transitive dependencies (kotlin-reflect, Glide),
  androidx.car.app 1.3.0, and the 8.0 API changes: removed
  DisplayOptions.showTrafficLights / showStopSigns and onGpsAvailabilityUpdate,
  new RouteStatus.ROUTING_TIMEOUT, terrain map type during navigation, and
  deprecations of ListenableResultFuture, SupportNavigationFragment,
  setAudioGuidance, FollowMyLocationOptions, and NavigationTransactionRecorder
  Waypoint overloads. Use when upgrading NavSDK to 8.0, fixing a build that
  broke after the bump, or auditing what 8.0 changes for an app. Don't use for
  iOS.
license: Apache-2.0
metadata:
  version: 1.0.0
---

# Migrating Navigation SDK for Android 7.x to 8.0

## Read this first: what does *not* change

This is a version bump of the **same** SDK. Don't rename packages, swap
artifacts, or change how the navigator is initialized.

*   **Package namespace stays `com.google.android.libraries.navigation.*`.** Do
    not rewrite imports.
*   **Artifact stays `com.google.android.libraries.navigation:navigation`.**
    Only the version changes.
*   **Initialization is unchanged.** Keep calling
    `NavigationApi.getNavigator(...)` exactly as before.
*   **`Navigator.RouteStatus` is still a Java `enum`.** Do not convert it to
    `int` constants.

## Workflow

Work in this order. Nothing compiles on 8.0 until the toolchain is right, and
fixing call sites against a build that won't configure wastes a cycle. Hard
breaks come before deprecations because removed symbols *cannot* compile, while
deprecated ones only warn. Phase 4 is a scheduling decision for the developer,
not a mandate.

### Phase 0: Preflight

Record these before you change anything; later phases branch on them:

*   Current NavSDK version
    (`com.google.android.libraries.navigation:navigation:<version>`). This skill
    covers 7.x to 8.0 only. If the app is on 6.x or older, first upgrade to the
    latest 7.x using the
    [release notes](https://developers.google.com/maps/documentation/navigation/android-sdk/release-notes),
    get a green build, then continue here.
*   Build DSL (Groovy `build.gradle` vs. Kotlin `build.gradle.kts`), AGP and
    Gradle versions, and the JDK running Gradle.
*   Source languages (Java, Kotlin, or both).
*   Whether layout XML references NavSDK fragments, and whether the app uses
    Android Auto (`androidx.car.app`).
*   The user's goal: full migration, just "make it build", or an audit only. For
    an audit, report the Phase 1-4 findings and make no edits.

Inventory the affected symbols in one pass. The result is the work list for the
rest of the migration: each hit maps to a Phase 1-4 item below, and an audit
reports these hits grouped by phase. Search layout XML too, because fragment
class names in XML only fail at inflation time. Kotlin code usually infers the
future type (`val pending = navigator.setDestination(...)`), so
`ListenableResultFuture` alone misses it; `setOnResultListener` catches those.

Search Java, Kotlin, XML, Gradle, and ProGuard files for the symbols below. Use
your environment's code search if there's no shell; with a shell, this command
does it:

```shell
grep -rnE "showTrafficLights|showStopSigns|onGpsAvailabilityUpdate|ListenableResultFuture|setOnResultListener|SupportNavigationFragment|setAudioGuidance\(|simulateLocationsAlongNewRoute|MAP_TYPE_TERRAIN|RouteStatus|getNavigator|NavigationTransactionRecorder|FollowMyLocationOptions|header[A-Za-z]*(TypefacePath|TextSize)|kotlin-reflect|kotlin\.reflect|bumptech\.glide|androidx\.car\.app" \
  --include=*.java --include=*.kt --include=*.xml --include=*.gradle --include=*.kts --include=*.pro .
```

### Phase 1: Toolchain

8.0's AAR metadata declares `minAndroidGradlePluginVersion=9.0.1`. On AGP 8.x,
Gradle **refuses to configure the build**: you get no deprecation warning, just
a hard failure. Fix this first.

If the project is **already on AGP >= 9.0.1** and compileSdk 37, Phase 1 is a
verification step only: confirm the Gradle wrapper satisfies that AGP version
and the JDK is 17/21, then move on. Don't downgrade a newer AGP to 9.0.1.

NavSDK 7.x builds fine on AGP 9.0.1 / Gradle 9.1.0 / compileSdk 37 (verified
with the published 7.9.0 sample apps). Bump the toolchain while still on 7.x,
get a green debug **and** release build, then change the NavSDK version. That
way every later failure is attributable to NavSDK, not AGP.

*   **Android Gradle plugin**: 8.13.2 to **9.0.1**. Hard minimum (AAR metadata).
*   **Gradle**: 8.13 to **9.1.0** (for AGP 9.0.1). Newer AGP 9.x needs newer
    Gradle (e.g. AGP 9.2.0 to Gradle 9.4.1); follow the AGP error message.
*   **`compileSdk` / `targetSdk`**: 36 to **37**. SDK targets 37.
*   **Kotlin**: 2.3.0 to 2.3.0. Unchanged.
*   **`desugar_jdk_libs_nio`**: 2.0.x to **2.1.5**. Enforced via AAR metadata.
*   **`com.google.gms:google-services`**: 4.4.x to **4.5.0**. Only if used (AGP
    9 removed `applicationVariants`).

AGP 9.0.1 prints "This Android Gradle plugin (9.0.1) was tested up to compile
SDK version 36.1" for compileSdk 37. That warning is expected and harmless; to
silence it add `android.suppressUnsupportedCompileSdk=37.0` to
`gradle.properties` (or use a newer AGP 9.x).

> [!CAUTION] Do **not** tell developers they need JDK 25. JDK 25 is the internal
> toolchain used to build NavSDK itself. For a host app on AGP 9, JDK 17 is the
> minimum Gradle runtime and JDK 21 LTS is recommended.

AGP 9 itself introduces behavior changes that surface as build failures with no
obvious NavSDK link (e.g. `package=` in the manifest, `kotlinOptions`,
`dexOptions`). Read [references/toolchain_bump.md](references/toolchain_bump.md)
for the symptom-to-fix list. Link the official
[AGP 9.0 release notes](https://developer.android.com/build/releases/agp-9-0-0-release-notes)
rather than restating them. For Kotlin Multiplatform modules that combine
`org.jetbrains.kotlin.multiplatform` with an Android plugin, follow JetBrains'
[AGP 9 migration guidance](https://kotlinlang.org/docs/kotlin-ai-skills.html#migrate-multiplatform-projects-with-android-apps-to-use-agp-9)
(the `kotlin-tooling-agp9-migration` skill) for the module restructuring, and
still apply the NavSDK floors above.

**targetSdk 37 has a Play Console consequence, not a code change:** apps that
use NavSDK must complete the **Location Purpose Declaration** in Google Play
Console. Select **Navigation** to justify persistent precise location
(`ACCESS_FINE_LOCATION`) during guidance, and also declare any non-navigation
uses. Tell the user; you cannot do it from code. See the
[Play Console location declaration guide](https://support.google.com/googleplay/android-developer/answer/17033915#play_console_declaration).

### Phase 2: Dependencies

8.0 stops pulling in several transitive dependencies. The size reduction depends
on the app: up to ~4.8 MB for apps that did not already strip them, and much
less for apps already shrinking with R8 (for example, full mode):

*   `org.jetbrains.kotlin:kotlin-reflect`
*   `com.github.bumptech.glide:glide` and
    `com.github.bumptech.glide:okhttp3-integration`

Add one back **only if the app's own code references it**: `kotlin.reflect.*`
APIs beyond `::class` / `KClass` basics, or `com.bumptech.glide.*` imports. A
blanket re-add undoes the size win NavSDK just shipped. A "class not found" for
these packages after the bump is the signal.

Conversely, if the app **explicitly declares** `kotlin-reflect` or Glide (often
copied in years ago to mirror NavSDK's transitive set) but has no
`kotlin.reflect.*` / `com.bumptech.glide.*` usage in its own code or ProGuard
rules, remove the declaration (and any matching `annotationProcessor` / `kapt` /
`ksp` entries) to realize the size win.

**Android Auto:** 8.0 no longer bundles ProGuard keep rules for
`androidx.car.app`. If the app uses the Android Auto projected mode, upgrade
`androidx.car.app:app` and `androidx.car.app:app-projected` to **1.3.0 or
higher** so their own consumer rules apply. Otherwise release (R8) builds can
strip required classes while debug builds still work.

Other library versions NavSDK bumped (AndroidX lifecycle 2.10.0, material
1.14.0, datastore 1.2.1, window 1.5.1, transport 4.1.1, etc.) resolve
automatically through Gradle. Act only if the app pins an older version and
dependency resolution fails.

### Phase 3: Hard breaks (must fix to compile or behave correctly)

*   `DisplayOptions.showTrafficLights(...)`, `showStopSigns(...)`: **Removed.**
    Delete the calls. Traffic lights and stop signs are now always shown during
    navigation in countries where this data is available.
*   `RoadSnappedLocationProvider.GpsAvailabilityEnhancedLocationListener.onGpsAvailabilityUpdate(boolean)`:
    **Removed.** Delete the override; move logic to
    `onGpsAvailabilityChange(GpsAvailabilityChangeEvent)`. A leftover
    `@Override` fails compilation.
*   `Navigator.RouteStatus.ROUTING_TIMEOUT`: **New enum constant: routing
    backend timed out.** Add a branch wherever the app handles every status (see
    below).
*   `GoogleMap.MAP_TYPE_TERRAIN` during navigation: **No longer supported.**
    Compiles fine; at runtime the map falls back to `MAP_TYPE_NORMAL` until
    navigation ends. Remove terrain options from in-navigation UI so users
    aren't offered a mode that silently doesn't apply.
*   `StylingOptions` `Parcelable` members (`CREATOR`, `writeToParcel`):
    **Internal, not part of the public API.** Don't parcel `StylingOptions`
    (e.g. via `Bundle` / `Intent` extras). Rebuild it from its builder methods
    where it's needed.
*   `NavigationApi.ErrorCode.NETWORK_ERROR` from `getNavigator(Application,
    ...)`: **Behavior change: can now fire.** In 7.x it was never reported. In
    8.0, when the terms check fails with a network/timeout error and there is no
    `Activity` (the `Application` overload), `onError` receives `NETWORK_ERROR`
    where it used to receive `TERMS_NOT_ACCEPTED`. If `onError` has no
    `NETWORK_ERROR` branch, add one that retries `getNavigator` (with backoff)
    instead of re-prompting for terms or treating it as fatal. Keep any existing
    `NETWORK_ERROR` handling.

**`ROUTING_TIMEOUT` and exhaustiveness.** The constant is additive, but it
breaks code that must be exhaustive:

*   A Kotlin `when (status)` used as an **expression** fails to compile until it
    covers `ROUTING_TIMEOUT` or has an `else`.
*   A Java `switch` **expression** (`->` form that yields a value) over the enum
    is a compile error too.
*   A plain Java `switch` statement compiles but silently skips the new status.
    Add a case so timeouts get user-visible handling (typically "retry").

```kotlin
val message = when (routeStatus) {
    Navigator.RouteStatus.OK -> null
    Navigator.RouteStatus.ROUTING_TIMEOUT -> "Route calculation timed out. Try again."
    Navigator.RouteStatus.NETWORK_ERROR -> "Network error. Check your connection."
    // ...existing branches...
    else -> "Could not calculate a route ($routeStatus)."
}
```

### Phase 4: Deprecations (compile with warnings in 8.0; fix now or schedule)

Present these to the user as a choice. Don't silently rewrite every call site
unless they asked for a full migration. For each one, report where it's used and
when it's removed. If the user asked for a full migration, or you are running
non-interactively, **migrate every deprecation that has a replacement** and list
the ones without a replacement (`StylingOptions` typeface/text-size, and
`FollowMyLocationOptions` when the app needs its custom options) as deferred.

*   `ListenableResultFuture<RouteStatus>` returned by
    `Navigator.setDestination(s)(...)`: replace with overloads taking a trailing
    `RouteDescriptionLoadCallback` and returning `void`. Removal: 9.0.
*   `Simulator.simulateLocationsAlongNewRoute(...)` returning
    `ListenableResultFuture`: replace with overloads taking a trailing
    `RouteDescriptionLoadCallback`. Removal: 9.0.
*   `SupportNavigationFragment`: replace with `NavigationFragment` (same
    package, same methods, `newInstance()` / `newInstance(GoogleMapOptions)`).
    Removal: 9.0.
*   `NavigationTransactionRecorder.pickup(Waypoint, List)` / `dropoff(Waypoint,
    List)`: replace with `pickup(List)` / `dropoff(List)`. Removal: 9.0.
*   `Navigator.setAudioGuidance(@AudioGuidance int)`: replace with
    `setAudioGuidanceSettings(AudioGuidanceSettings)`. Removal: ~1 year after
    8.0.
*   `StylingOptions` typeface-path and text-size methods (`header*TypefacePath`,
    `header*TextSize` and getters): no replacement; color options remain.
    Removal: Not announced.
*   `GoogleMap.followMyLocation(int, FollowMyLocationOptions)` and the
    `FollowMyLocationOptions` class (`com.google.android.gms.maps.model`): no
    direct replacement. If the app doesn't need the custom options (e.g. zoom
    level), call `followMyLocation(int)` with the same perspective; that
    overload is not deprecated. Otherwise keep the call and report it as
    deferred. Removal: a future release (not yet announced).

#### `ListenableResultFuture` to `RouteDescriptionLoadCallback`

`RouteDescriptionLoadCallback` is the top-level interface
`com.google.android.libraries.navigation.RouteDescriptionLoadCallback` with a
single method, `onRouteDescription(RouteDescription)`, invoked on the UI thread.
`RouteDescription.getRouteStatus()` returns the **enum**
`Navigator.RouteStatus`.

```java
// Before (deprecated in 8.0)
ListenableResultFuture<Navigator.RouteStatus> future = navigator.setDestination(waypoint);
future.setOnResultListener(status -> {
  if (status == Navigator.RouteStatus.OK) {
    navigator.startGuidance();
  }
});

// After
navigator.setDestination(
    waypoint,
    routeDescription -> {
      if (routeDescription.getRouteStatus() == Navigator.RouteStatus.OK) {
        navigator.startGuidance();
      }
    });
```

```kotlin
navigator.setDestination(waypoint) { routeDescription ->
    if (routeDescription.routeStatus == Navigator.RouteStatus.OK) {
        navigator.startGuidance()
    }
}
```

Each `setDestination` / `setDestinations` overload (with `RoutingOptions`,
`DisplayOptions`, `CustomRoutesOptions`) has a callback twin with the same
leading parameters.

#### `SupportNavigationFragment` to `NavigationFragment`

Replace the class in **both** code and layout XML, covering both the
`android:name="..."` and `class="..."` attributes on `<fragment>` /
`FragmentContainerView`. A code-only replacement compiles cleanly and then
crashes at inflation, or keeps the deprecated class alive in XML.

```xml
<fragment
    android:id="@+id/navigation_fragment"
    android:name="com.google.android.libraries.navigation.NavigationFragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

`NavigationFragment` is an AndroidX `Fragment`, like
`SupportNavigationFragment`, so `supportFragmentManager` usage is unchanged.
Update every reference, not just imports: field/property types, casts such as
`(SupportNavigationFragment) getSupportFragmentManager().findFragmentById(...)`,
`instanceof` / Kotlin `is` checks, `SupportNavigationFragment()` constructor
calls, `SupportNavigationFragment.newInstance(...)` calls, and KDoc/Javadoc
`[SupportNavigationFragment]` links. See
[Update to NavigationFragment](https://developers.google.com/maps/documentation/navigation/android-sdk/nav-map-interactions-best-practices#update-to-navigation-fragment).

#### `setAudioGuidance(int)` to `setAudioGuidanceSettings(...)`

The defaults are **inverted**. The old flag bitmask was additive from zero, but
`AudioGuidanceSettings.builder()` starts with vibration on,
`GuidanceMode.VOICE_ALERTS_AND_GUIDANCE`, Bluetooth audio on, and
`VolumeLevel.NORMAL`. Explicitly turn off anything the old bitmask omitted.

Map each `Navigator.AudioGuidance` flag to an `AudioGuidanceSettings.Builder`
call:

*   `SILENT`: `setGuidanceMode(GuidanceMode.SILENT)`
*   `VOICE_ALERTS_ONLY`: `setGuidanceMode(GuidanceMode.VOICE_ALERTS_ONLY)`
*   `VOICE_ALERTS_AND_GUIDANCE`:
    `setGuidanceMode(GuidanceMode.VOICE_ALERTS_AND_GUIDANCE)`
*   `VIBRATION` set / absent: `setVibrationEnabled(true)` /
    `setVibrationEnabled(false)`
*   `BLUETOOTH_AUDIO` set / absent: `setBluetoothAudioEnabled(true)` /
    `setBluetoothAudioEnabled(false)`

```java
// Before: navigator.setAudioGuidance(
//     Navigator.AudioGuidance.VOICE_ALERTS_AND_GUIDANCE | Navigator.AudioGuidance.BLUETOOTH_AUDIO);
navigator.setAudioGuidanceSettings(
    AudioGuidanceSettings.builder()
        .setGuidanceMode(AudioGuidanceSettings.GuidanceMode.VOICE_ALERTS_AND_GUIDANCE)
        .setBluetoothAudioEnabled(true)
        .setVibrationEnabled(false) // VIBRATION was not in the old bitmask
        .build());
```

### Phase 5: Verify and report

1.  Build **both** debug and release variants. R8-only failures (Android Auto
    keep rules, prohibited consumer keep rules under AGP 9) appear only in
    release.
2.  Surface deprecation warnings: Kotlin prints `w:` lines by default; for Java
    add `options.compilerArgs += ["-Xlint:deprecation"]` to
    `tasks.withType(JavaCompile)` (or read the per-file "uses or overrides a
    deprecated API" note). Re-run the Phase 0 grep. Removed symbols must be
    gone. Any remaining deprecated usages must be ones the user chose to defer.
3.  Give the user a short report:
    *   What changed (toolchain, dependencies, call sites).
    *   Deprecations deferred, with file locations and their removal version
        (9.0, ~1 year, or a future release).
    *   Non-code follow-ups: the Play Console Location Purpose Declaration, and
        optionally the new off-route traffic incident cards, which are **on by
        default** when the traffic layer is enabled. They can be disabled with
        `setTrafficIncidentCardsEnabled(false)` on `NavigationView` /
        `NavigationFragment` if the app's UX needs that.

## Verification checklist

-   [ ] AGP >= 9.0.1 with the Gradle version that AGP requires, Gradle JDK >=
    17; `compileSdk` / `targetSdk` 37; NavSDK dependency at 8.0.x.
-   [ ] Imports still `com.google.android.libraries.navigation.*`;
    `NavigationApi.getNavigator(...)` initialization unchanged; `RouteStatus`
    still used as an enum.
-   [ ] No references to `showTrafficLights`, `showStopSigns`, or
    `onGpsAvailabilityUpdate`.
-   [ ] Exhaustive `RouteStatus` handling covers `ROUTING_TIMEOUT`;
    `RouteStatus.NETWORK_ERROR` handling retained.
-   [ ] `getNavigator` `onError` handles `ErrorCode.NETWORK_ERROR` with a retry
    (it can now fire from `getNavigator(Application, ...)`).
-   [ ] `kotlin-reflect` / Glide declared only if the app uses them directly;
    `androidx.car.app` >= 1.3.0 if Android Auto is used.
-   [ ] `StylingOptions` is not parcelled; terrain isn't offered during
    navigation.
-   [ ] Deprecations either migrated (including `SupportNavigationFragment` in
    layout XML) or listed in the report as deferred.
-   [ ] Debug **and** release builds succeed; user reminded about the Play
    Location Purpose Declaration.
