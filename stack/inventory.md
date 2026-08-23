# Stack Inventory

The components of the Sector system, their repositories, technology, status, and
where to find build/test entry points. All technology facts below were gathered by
**read-only** inspection of the local checkouts on 2026-08-22. Where a fact could
not be verified read-only, it is marked _(unverified)_.

## Summary table

| Component | Repository / service | Responsibility | Initial status |
|---|---|---|---|
| Orchestrator | https://github.com/mgcather07/Sector_Orchestrator.git | Cross-platform contracts and coordination | Being established |
| iOS | https://github.com/mgcather07/iOS_Sector.git | Main behavioral source of truth and native iOS app | Existing active application (v4.0.8) |
| Android | https://github.com/mgcather07/Android_Sector.git | Android implementation | Existing, **substantial** (~83k LOC, v4.0.6) |
| Web | https://github.com/mgcather07/Web_Sector.git | Web implementation | Existing, **mature cross-role app** |
| Firebase Realtime Database | Hosted Firebase (`sector-9393c`) | Shared application database | Existing; schema audited from rules, full field audit pending |
| Sector_Engine | https://github.com/mgcather07/Sector_Engine.git (Cloud Run) | Shared bowfishing "shooting-conditions" scoring service | Existing; **out of scope for this bootstrap** |
| BAA records backend | Firebase project `baa-website-908e4` (REST) | External big-fish record database | Existing; iOS/Android talk to it via REST |

> Firebase project: **`sector-9393c`**. Two RTDB instances — prod
> `sector-9393c-default-rtdb`, dev `sector-dev-1f4d0` — with a runtime master-only
> dev/prod switch on iOS and Android, and dual `build:dev`/`build:prod` on Web.

---

## Orchestrator

- **Responsibility:** Cross-platform feature contracts, platform scope, shared
  domain & RTDB contracts, decisions, migrations, parity assessments, tasks.
- **Local path:** `/Users/mcather/Desktop/Development/Orchestrator/Sector`
- **Technology:** Markdown + small YAML metadata blocks under git.
- **Status:** Being established (this bootstrap).
- **Schema responsibility:** Documents the shared RTDB contract; owns no data.
- **Deployment responsibility:** None.
- **Build/test:** None.
- **Open questions:** Local paths on the Mac mini for a canonical layout (see
  [`../docs/open-questions.md`](../docs/open-questions.md) Q1).

## iOS — main source of truth

- **Responsibility:** Native iOS app; the reference for current product behavior and
  current RTDB usage. Primarily developed by Michael.
- **Local path:** `/Users/mcather/Desktop/Development/iOS/Sector`
- **Technology (confirmed):** Swift, SwiftUI, **iOS 17.4+** (`IPHONEOS_DEPLOYMENT_TARGET
  = 17.4`), **SwiftData** (19 `@Model` classes in `SectorSchemaV1`), **Firebase RTDB**,
  Firebase Auth, Firebase Cloud Messaging, Firebase App Check, Firebase Remote Config,
  **MapKit**, **StoreKit 2** (direct — no RevenueCat), **Kingfisher 8.2.0**,
  Google Sign-In, CryptoKit, EventKit, WatchConnectivity, CoreLocation.
  `firebase-ios-sdk 10.29.0` via SPM (no CocoaPods). Version **4.0.8 (25)**, bundle
  `io.sector.co`.
- **Refuted:** **Firestore** (0 usages; RTDB-only), **RevenueCat** (root file is a
  1-byte empty placeholder; StoreKit 2 is used directly), **Mapbox** (dead code, not
  linked; MapKit is the map, Apple search is used for addresses).
- **Architecture:** MVVM + singleton managers/services (`DatabaseProvider`,
  `DataPuller`, `FirebaseListener`, `StoreKitManager`, `NotificationManager`,
  `ZoneManager`, `CurrentUser`, `UserService`). Entry `SectorApp.swift` →
  `AppDelegate.swift` → `RootView.swift`. Companion **SectorWatch** app (drop markers
  via WatchConnectivity).
- **Status:** Existing active application; the behavioral reference.
- **Schema responsibility:** Defines/represents the current RTDB usage. Ships
  `database.rules.json` and `functions/` (the deployed Cloud Functions live here).
- **Deployment responsibility:** App Store (TestFlight scripts in `scripts/`);
  co-owner of the deployed RTDB rules + Cloud Functions.
- **Build/test (confirmed):** Schemes `Sector`, `Sector-Dev`.
  `xcodebuild -scheme Sector -destination 'generic/platform=iOS Simulator' build`;
  `xcodebuild test -scheme Sector -destination 'platform=iOS Simulator,name=iPhone 16'`.
  Tests in `SectorTests/` (XCTest) and `SectorUITests/`.
- **Parity docs:** `specs/` (~57 files, `INVENTORY.md` = 1237 lines), `docs/handoff/`
  (dated up to **2026-08-06**), `docs/decisions/0001-runtime-dev-prod-database.md`.
- **Notes:** Root `AGENTS.md`/`CLAUDE.md` are **stale** (understate the app, wrong
  version) — trust `specs/` and source. Trips and custom markers are **local-only**
  (SwiftData), except team-shared markers (`sharedMarkers`).

## Android

- **Responsibility:** Android implementation. Source of truth for the Android app only.
- **Local path:** `/Users/mcather/Desktop/Development/Android/SectorAndroid`
- **Technology (confirmed):** **Kotlin**, **Jetpack Compose** (Material 3), single-Activity
  + `navigation-compose`, **minSdk 31 / target 36 / compile 36**, JDK 17. Firebase BoM
  **34.12.0** (auth, database, storage, messaging, analytics, crashlytics, App Check
  Play Integrity), **Google Play Billing 7.1.1**, **Google Maps** (`maps-compose`) +
  fused location, **Room 2.6.1** (local cache), **Coil**, OkHttp, DataStore, `bouquet`
  (PDF). MVVM with **manual DI** (ViewModel factories, no Hilt). Version **4.0.6 (45)**.
- **Maturity (high confidence):** **Substantial** — 354 Kotlin files, ~82,850 LOC, 1
  `TODO`, 33 unit-test files. Signed for Play release. Has matured **past** its own
  2026-07-15 parity audit (Chum Bucket, redzone submission now in source) and includes
  features **beyond** the iOS spec set (bank spots, boat tracks, my-lakes, dam-generation).
- **Status:** Existing, substantial, near-parity with active parity program.
- **Schema responsibility:** **None** — `database.rules.json`, `functions/`,
  `.firebaserc`, `firebase.json` are **absent** in-repo (owned by the backend/iOS repo).
  Android must mirror iOS RTDB field names exactly; all access routes through
  `utils/DatabaseProvider`.
- **Deployment responsibility:** Google Play (`bundleRelease` → AAB). Keystore present
  (contents not read).
- **Build/test (confirmed from docs, not run):** `./gradlew assembleDebug`,
  `./gradlew compileDebugKotlin`, `./gradlew testDebugUnitTest`, `./gradlew bundleRelease`.
- **Parity docs:** `specs/` (55 `.md`, a **generated mirror** of iOS specs via
  `ops/sync-specs.sh` — do not hand-edit), `docs/PARITY-AUDIT.md` (2026-07-15, ~880
  lines, the key artifact), `docs/iOS-PARITY-ROADMAP.md` (2026-05-29, older/stale).
  `tasks/` empty; `ops/` deploy pipeline parked/unused.
- **Notes / flag:** **Background geofencing is dormant** — `GeofenceManager` code exists
  but `AndroidManifest.xml` does not declare `ACCESS_BACKGROUND_LOCATION`, so
  `refresh()` no-ops and zone alerts run **foreground-only**. Roadmap's "Notifications
  100%" overstates this. See [`../parity/unresolved-conflicts.md`](../parity/unresolved-conflicts.md).

## Web

- **Responsibility:** Web implementation. Source of truth for the Web app only.
- **Local path:** `/Users/mcather/Desktop/Development/Web/Sector-Web`
- **Technology (confirmed):** **Next.js 14.2** (App Router, all pages `"use client"`),
  **React 18.3**, **TypeScript 5.5**, **Tailwind 3.4**, **firebase v11** (Auth, RTDB,
  Storage, Analytics/GA4), **Mapbox GL 3.8** + Mapbox GL Draw + Mapbox Geocoding.
  **Static export** (`output: "export"` → `out/`) deployed to **Firebase Hosting**
  (dev + prod targets). No Redux; React Context for auth + `onValue` hooks.
- **Maturity (high confidence):** **Mature, multi-role** customer + premium + owner +
  admin/master app (~8,900 LOC pages + ~3,800 LOC lib, 33 `page.tsx` routes). Serves
  **both customers and administrators**. README's "admin redzone tool" framing is
  **stale** — trust the code.
- **Status:** Existing, mature.
- **Schema responsibility:** Ships its **own copy** of `database.rules.json` (deployed
  to both dev+prod via `firebase.json`) and a `functions/` codebase (`appstore`). This
  copy has **drifted** from the iOS repo's copy — see
  [`../parity/unresolved-conflicts.md`](../parity/unresolved-conflicts.md).
- **Deployment responsibility:** Firebase Hosting (`deploy:dev`/`deploy:prod`); co-owner
  of deployed RTDB rules + a Cloud Functions codebase.
- **Build/test (confirmed):** `npm run dev`, `next build`, `build:dev`/`build:prod`,
  `deploy:dev`/`deploy:prod`, `next lint`. **No automated test runner** (no Jest/Vitest/Playwright).
- **Notes:** **No web payments** — subscriptions are mobile-store only; Web *unlocks*
  premium via `entitlements/{uid}` or `users/{uid}/plan`. Cloud Functions here include
  `appStoreNotifications`, `playNotifications`, `notifyRedzoneSubmission*`,
  `adminListUsers`, `clearPastTournamentRedzoneLinks`, `syncBaaRegistrations`, and the
  `conditions` engine.

## Firebase Realtime Database (shared backend)

- **Responsibility:** The shared application database for all clients.
- **Project:** `sector-9393c`. Instances: prod `sector-9393c-default-rtdb`, dev
  `sector-dev-1f4d0`.
- **Contract:** 58 top-level nodes — see [`../contracts/realtime-database.md`](../contracts/realtime-database.md).
- **Rules/functions ownership:** Rules are checked into **both** the iOS repo and the
  Web repo and deployed from there; Cloud Functions live in iOS `functions/` and Web
  `functions/`. **This dual ownership is a governance risk** (rules have drifted) — see
  open questions Q19 and the unresolved conflicts register.
- **Storage:** Firebase Storage (`sector-9393c.appspot.com`) for images
  (`ProfilePics/`, `CoverPics/`, `Stores/`, `Guides/`, `Tournaments/`).
- **Change control:** Any RTDB change is a gated cross-platform change (see AGENTS.md §9).

## Sector_Engine (external, out of scope)

- **Responsibility:** Pure-Swift service on Google Cloud Run computing bowfishing
  shooting-conditions scores, tuned via Firebase Remote Config, so iOS and Android
  render identical numbers. Clients: iOS `EngineAPIClient.swift`; Android
  `conditions/`.
- **Status:** Existing. **Not inspected** in this bootstrap; not modified.

## BAA records backend (external)

- **Responsibility:** Big-fish record database in a **separate** Firebase project
  `baa-website-908e4`, accessed by iOS/Android over **plain REST** (not the SDK).
- **Status:** Existing; referenced by the records features (`baa-record-submit`,
  `baa-records-browser`, `record-claims`).
