# Platform: iOS

- **Repository:** https://github.com/mgcather07/iOS_Sector.git
- **Local path:** `/Users/mcather/Desktop/Development/iOS/Sector`
- **Product role:** Native iOS app; primary customer surface. Primarily developed by Michael.
- **Source-of-truth responsibility:** **Main source of truth for current product
  behavior and current Realtime Database usage.** Other platforms build toward iOS
  behavior (as scoped by orchestrator contracts).
- **Current status:** Existing active application, **v4.0.8 (build 25)**, bundle `io.sector.co`.

## Technology (confirmed 2026-08-22)

- Swift + SwiftUI, **iOS 17.4+** (`IPHONEOS_DEPLOYMENT_TARGET = 17.4`).
- **SwiftData** — `SectorSchemaV1` versioned schema with a migration plan; 19 `@Model`
  classes (`User, Redzone, Store, Guide, Zone, Tournament, Sponsor, Address, MapMarker,
  Regulation, Format, RegisteredTeam, Fish, Series, Trip, FishEntry`, …). `BoatTrack` in
  a separate container.
- **Firebase RTDB** (backing store), Firebase Auth, Firebase Cloud Messaging, Firebase
  App Check, Firebase Remote Config. `firebase-ios-sdk 10.29.0` via SPM.
- **MapKit** (SwiftUI `Map`, MapKit search/geocoding). **StoreKit 2** (direct).
  **Kingfisher 8.2.0** (images). Google Sign-In, CryptoKit, EventKit, WatchConnectivity,
  CoreLocation.
- **Not used:** Firestore (RTDB-only), RevenueCat (empty placeholder), Mapbox (dead code).

## Architecture

- Entry `SectorApp.swift` (`@main`, SwiftData container) → `AppDelegate.swift` (App Check
  → `FirebaseApp.configure()` → Messaging) → `RootView.swift` (auth gate/root nav).
- MVVM + singleton managers/services: `DatabaseProvider` (routes all DB access),
  `DataPuller` (initial fetch), `FirebaseListener`/`Listener` (live `child` observers),
  `StoreKitManager`, `NotificationManager`, `ZoneManager`, `CurrentUser`, `UserService`.
- Source under `Sector/`: `UI/` (per feature area), `Filter/`, `Notifications/`,
  `Store Kit/`, `SwiftData/`, `Utils/Providers/`, `Custom Views/`.
- Companion **SectorWatch** app (drop map markers via WatchConnectivity).
- Companion external service **Sector_Engine** (Cloud Run) computes conditions scores.

## Capabilities & constraints

- Background geofencing via `CLLocationManager` region monitoring (the reference impl).
- Trips and personal markers are **local-only** (SwiftData); only `sharedMarkers` sync.
- BAA records use a **separate** Firebase project (`baa-website-908e4`) over REST.
- Two RTDB instances (prod/dev) with a master-only runtime switch; Debug defaults to dev.

## Build & test (confirmed)

- Schemes: `Sector`, `Sector-Dev`.
- Build: `xcodebuild -scheme Sector -destination 'generic/platform=iOS Simulator' build`
- Test: `xcodebuild test -scheme Sector -destination 'platform=iOS Simulator,name=iPhone 16'`
- Tests: `SectorTests/` (XCTest unit), `SectorUITests/` (UI). Release scripts in `scripts/`.

## Distribution / deployment

- App Store / TestFlight (`scripts/release-testflight.sh`, `ExportOptions.plist`).
- **Co-owns the deployed RTDB rules** (`database.rules.json`), **Cloud Functions**
  (`functions/index.js`, ~30), and `storage.rules`. See conflict C4 (rules drift with Web).

## Ownership

- Michael (primary developer). Changes here are the behavioral reference — avoid modifying
  iOS during Android/Web work unless a task authorizes it.

## Open questions

- Root `AGENTS.md`/`CLAUDE.md` are stale — should they be refreshed to match reality?
- Sponsor model intended usage (conflict C6).
- Which repo should own RTDB rules + Cloud Functions long-term (Q19 / conflict C4)?
