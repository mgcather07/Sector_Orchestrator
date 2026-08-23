# Platform: Android

- **Repository:** https://github.com/mgcather07/Android_Sector.git
- **Local path:** `/Users/mcather/Desktop/Development/Android/SectorAndroid`
- **Product role:** Native Android app; customer surface, built toward iOS behavior.
- **Source-of-truth responsibility:** Source of truth for the **current Android
  implementation only** — not for shared product intent.
- **Current status:** Existing and **substantial** — 354 Kotlin files, ~82,850 LOC, 1
  `TODO`, signed for Play release. **v4.0.6 (build 45).** An active parity program.

## Technology (confirmed 2026-08-22)

- **Kotlin**, **Jetpack Compose** (Material 3), single-Activity + `navigation-compose`.
- **minSdk 31 / targetSdk 36 / compileSdk 36**, JDK/jvmTarget **17**. (Prose docs saying
  "target 35" are stale — gradle is authoritative.)
- Firebase BoM **34.12.0**: auth, database (RTDB), storage, messaging (FCM), analytics,
  crashlytics, **App Check (Play Integrity)**.
- **Google Play Billing 7.1.1** (`sector.premium.yearly`). **Google Maps** (`maps-compose`)
  + fused location. **Room 2.6.1** (local cache, KSP). **Coil** (images). OkHttp,
  DataStore, `bouquet` (PDF).
- MVVM with **manual DI** (ViewModel factories; no Hilt). RTDB access centralized in
  `utils/DatabaseProvider` (prod/dev switch, disk persistence, master env toggle).
- Mapbox libs present as deps but the redzone map renders with Google Maps.

## Maturity (high confidence)

- **Substantial, not a skeleton.** Full Room layer (15 DAOs, 21 models), real engines
  (tournament results/side-pots, conditions, trip-recall, geofence math, billing), 33
  unit-test files.
- Has matured **past** its own 2026-07-15 `PARITY-AUDIT.md`: Chum Bucket
  (`tournament/live/ChumBucket.kt`, `ChumDrawScreen.kt`) and redzone submission
  (`redzones/RedzoneSubmitScreen.kt`, `RedzoneDraftService.kt`) are now in source.
- Includes features **beyond** the iOS spec set: bank spots (`bank/`), boat tracks
  (`tracks/`), my-lakes (`mylakes/`), dam-generation (`conditions/generation/`).

## Capabilities & constraints

- **Background geofencing is dormant** — `GeofenceManager` code exists but
  `AndroidManifest.xml` lacks `ACCESS_BACKGROUND_LOCATION`, so it no-ops and zone alerts
  are **foreground-only**. See conflict C1. (Contradicts the roadmap's "100%".)
- **Apple Sign-In intentionally absent** (Google + email + guest only).
- **Ships no backend contract:** `database.rules.json`, `functions/`, `.firebaserc`,
  `firebase.json` are absent — Android **mirrors iOS RTDB field names exactly** and must
  never rename a synced field.
- Conditions scoring is a deliberately simplified reimplementation; scores may diverge
  from iOS/Sector_Engine on identical inputs (documented divergence).

## Build & test (from docs; not run)

- `./gradlew assembleDebug` · `./gradlew compileDebugKotlin`
- `./gradlew testDebugUnitTest` (tests in `app/src/test/` — 33 files)
- `./gradlew bundleRelease` (AAB → Play Console). Single module `app`, Kotlin DSL, JDK 17.

## Distribution / deployment

- Google Play. Keystore present (`keystore/`, `keystore.properties` — contents not read).
- App Check keys in per-variant `app/src/{release,debug}/…/AppCheck.kt` (not read).

## Ownership

- Maintained through **approved development tasks** (task-driven autonomy). Michael reviews.

## Existing parity docs

- `specs/` (55 md) — a **generated mirror** of iOS specs via `ops/sync-specs.sh`
  (do not hand-edit).
- `docs/PARITY-AUDIT.md` (2026-07-15) — the key artifact (now partly stale).
- `docs/iOS-PARITY-ROADMAP.md` (2026-05-29) — older phase roadmap (stale).
- `tasks/` currently empty; `ops/` deploy pipeline parked/unused.

## Open questions

- Was the tournament-browse state/sort/50-mi filter wiring completed, or still dead code? (pilot)
- Background geofencing: enable + verify, or accept foreground-only? (C1)
- Adopt the shared Sector_Engine conditions numbers, or accept divergence?
- Are Firebase Analytics events (present on iOS spec) required on Android?
- Update no-name fallback to "Member" (C3).
