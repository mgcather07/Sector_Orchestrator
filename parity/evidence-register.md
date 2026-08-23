# Evidence Register

Where the conclusions in this orchestrator's parity/contract docs come from. Paths are
repo-relative to each platform checkout. This register points at evidence; it does not
copy source files or full specs. Gathered by read-only inspection on **2026-08-22**.

Reliability: **primary** = current source/rules read directly · **secondary** =
maintained spec/audit · **tertiary** = older plan/roadmap or inference.

## iOS (`iOS_Sector`, local `iOS/Sector`)

| Evidence path | Type | Supports | Reliability |
|---|---|---|---|
| `database.rules.json` | RTDB rules (25KB) | 58-node RTDB contract, indexes, gates | primary |
| `SectorApp.swift`, `AppDelegate.swift`, `RootView.swift` | app entry | Architecture, FCM token, App Check ordering | primary |
| `Sector/SwiftData/` (`DataPuller`, `Listener`, `ModelConversions`, `Models/`) | data layer | 19 `@Model` classes, RTDB↔SwiftData mapping, local-only trips/markers | primary |
| `Sector/Store Kit/StoreKitManager.swift`, `StoreKitConnect.storekit` | StoreKit 2 | Subscriptions (direct StoreKit, no RevenueCat), product IDs, upgrade-only | primary |
| `Sector/Notifications/`, `ZoneManager.swift`, `Zone+Geofencing.swift` | notifications | FCM topics, local reminders, background geofencing | primary |
| `Sector/UI/UserAuth/` (`Login`, `SignUp`, `CurrentUser`) | auth | Email/Google/Apple/guest, live role listener | primary |
| `functions/index.js` | Cloud Functions (~30) | Server triggers, `publicUsers` mirror, entitlements | primary |
| `.firebaserc`, `firebase.json` | Firebase config | Project `sector-9393c`, prod/dev instances | primary |
| `Sector.xcodeproj/project.pbxproj`, `…/Package.resolved` | build config | Swift/SwiftUI, iOS 17.4+, deps, v4.0.8(25) | primary |
| `specs/` (~57 files, `INVENTORY.md` 1237 lines, `README.md`) | spec set | Feature catalog, parity intent, terminology rules | secondary |
| `docs/handoff/*` (to 2026-08-06), `docs/decisions/0001-runtime-dev-prod-database.md` | handoffs/ADR | Recent iOS changes, dev/prod DB decision | secondary |
| `README.md` | root doc | Sector_Engine (Cloud Run) conditions service | secondary |
| root `AGENTS.md`, `CLAUDE.md` | working docs | **Stale** — understate the app; use with caution | tertiary |

## Android (`Android_Sector`, local `Android/SectorAndroid`)

| Evidence path | Type | Supports | Reliability |
|---|---|---|---|
| `app/build.gradle.kts` | build config | Kotlin/Compose, minSdk 31/target 36, deps, Play Billing, v4.0.6(45) | primary |
| `app/src/main/AndroidManifest.xml` | manifest | **No `ACCESS_BACKGROUND_LOCATION`** → geofence dormant | primary |
| `app/src/main/java/io/sector/co/` (354 files) | source | Feature presence/depth; RTDB via `DatabaseProvider` | primary |
| `notifications/GeofenceManager.kt`, `ZoneManager.kt` | geofencing | Background code present but gated off by manifest | primary |
| `subscription/*` (`BillingViewModel`, `SubscriptionPlan`) | billing | Play Billing `sector.premium.yearly`, `computeHasPremium` | primary |
| `tournament/live/ChumBucket.kt`, `ChumDrawScreen.kt`; `redzones/RedzoneSubmitScreen.kt`, `RedzoneDraftService.kt` | source | Features present **beyond** the 2026-07-15 audit | primary |
| `app/src/test/` (33 files) | unit tests | Conditions/geo/mention logic coverage | primary |
| `docs/PARITY-AUDIT.md` (2026-07-15, ~880 lines) | audit | Per-feature status + file:line + "shipped this pass" | secondary |
| `docs/iOS-PARITY-ROADMAP.md` (2026-05-29) | roadmap | Phase plan; Google Maps decision; **stale** | tertiary |
| `specs/` (55 md, generated mirror) + `ops/sync-specs.sh` | spec mirror | Same specs as iOS; sync mechanism | secondary |
| root `AGENTS.md`, `CLAUDE.md` | working docs | Parity intent, `DatabaseProvider` rule; some **stale** (target 35 vs 36) | tertiary |
| _Absent:_ `database.rules.json`, `functions/`, `.firebaserc` | — | Confirms rules/functions owned by backend/iOS repo | primary |

## Web (`Web_Sector`, local `Web/Sector-Web`)

| Evidence path | Type | Supports | Reliability |
|---|---|---|---|
| `package.json`, `next.config.mjs`, `tsconfig.json` | build config | Next.js 14 App Router, static export, deps, scripts | primary |
| `database.rules.json` (deployed prod+dev) | RTDB rules | Web's rules copy; **drift vs iOS copy** | primary |
| `src/app/**/page.tsx` (33 routes) | source | Mature cross-role app (customer + admin/master) | primary |
| `src/lib/` (`useAuth.tsx`, `data.ts`, `redzones.ts`, `weighIn.ts`, `types.ts`, …) | data/domain | Auth providers, RTDB hooks, domain types | primary |
| `src/components/PublicMap.tsx` + Mapbox deps | map | Mapbox GL + Draw (not Google/Leaflet) | primary |
| `functions/src/` (`appStoreNotifications`, `playNotifications`, `conditions`, `notifyRedzoneSubmission*`, `adminListUsers`, …) | Cloud Functions | Entitlements webhooks, conditions engine, admin listing | primary |
| `docs/redzone-approval-contract.md` (2026-06-11) | contract | Shared redzone submit→approve lifecycle | secondary |
| `functions/README.md` | doc | StoreKit → `entitlements` webhook | secondary |
| `README.md` | root doc | **Stale** "admin tool" framing — trust code | tertiary |
| `.firebaserc`, `firebase.json` | config | Project/hosting/db targets (prod+dev) | primary |

## Cross-cutting evidence

| Evidence | Supports | Reliability |
|---|---|---|
| iOS `database.rules.json` vs Web `database.rules.json` (diffed) | RTDB rules **drift** (`playAccountTokens`/`offerCodes`, entitlement writes, `seriesTeamId` index) | primary |
| iOS `specs/` vs Android `specs/` (diffed) | Identical except iOS-only `lake-conditions-alerts.md` | primary |
| iOS spec terminology note (`INVENTORY.md`) | "Member" no-name fallback (2026-07-19) supersedes "Bowfisher" | secondary |
