# Contract: Domain Model

**Status:** Draft (initial catalog). Field-level details require a full audit before
this is canonical. **Do not invent fields** — un-cited fields are marked as assumed.

Shared domain concepts and how each platform represents them. The goal is a common
vocabulary, not a shared class hierarchy. iOS is the behavioral reference; iOS uses
**SwiftData** `@Model` classes locally, Android uses **Room** entities + per-feature
models, Web uses **TypeScript types** in `src/lib/types.ts`. The **RTDB path** is the
shared surface (see [`realtime-database.md`](realtime-database.md)).

## App-wide terminology rule

> Sector is a **bowfishing** app. Users are **bowfishers**, never "anglers."
> **No-name display fallback = "Member"** (changed from "Bowfisher" on **2026-07-19**
> on iOS — the user category alongside Guide / Store). Never "Angler", "Unknown User".
> **Do not rename code identifiers / data contracts** that contain "angler" — e.g.
> `AppMode.angler` and the `anglerNames` booking field stay as-is; only user-facing
> *text* changes. (Android parity docs still reference the older "Bowfisher" fallback —
> see [`../parity/unresolved-conflicts.md`](../parity/unresolved-conflicts.md).)

## Concept catalog

For each concept: **iOS** (SwiftData model / representation) · **Shared facts** ·
**Assumptions** · **Android** · **Web** · **RTDB path** · **Open questions**.

### Tournament
- **iOS:** `Tournament.swift` (`@Model`). Rich record: dates, address ref, formats
  (`formatIds`), fee, registration toggle + opens-at, side pots, `ownerUserId`,
  `coHostUserIds` (map), series link.
- **Shared facts:** Public read; owner/co-host/master write. Dates stored as **Unix
  seconds**. Sorted by `startDate`. `isCurrentOrUpcoming` visibility rule.
- **Android:** `db/models/Tournament.kt` + `tournament/`. Confirmed present.
- **Web:** `src/lib/types.ts` `Tournament`, `/tournaments`, `/tournament`.
- **RTDB:** `tournaments/{tid}` (indexed `ownerUserId`); image `Tournaments/{id}.jpg`.
- **Open:** full field list & side-pot sub-shape; time component on dates (iOS date+time
  vs Android historically date-only).

### Redzone
- **iOS:** `Redzone.swift`. Geofenced area rendered as **polygon / line / point**
  (branch on `resolvedMapFeatureType`, default polygon). Has `sponsorId`, zone type,
  optional linked tournament, center/radius (derived).
- **Shared facts:** Public read; master or linked-tournament-owner write. Draft →
  submission → master approval → publish. Zone type drives accent color; type 7
  (Hidden) suppressed.
- **Android:** `redzones/` (Google Maps overlays); `RedzoneSubmitScreen.kt`.
- **Web:** `/map`, `/redzones`, `/redzone-requests`, `/review` (Mapbox GL Draw).
- **RTDB:** `redzones/{rzId}`, `redzoneDrafts/{uid}`, `redzoneSubmissions/{sid}`.
- **Open:** exact geometry field names; sponsor resolution.

### Map marker (personal)
- **iOS:** `MapMarker.swift` (`@Model`). **Local-only** (SwiftData). Name/date/color/
  type/notes; free = 25 + 3 colors, premium = ∞ + 6 colors.
- **Shared facts:** Personal markers are **not synced** to RTDB except when shared to a
  team. Cross-device favorites/markers sync exists via a user-data sync service
  (premium-gated) writing full marker payloads.
- **Android:** `markers/` (create/edit/list, guest markers). **Web:** type referenced;
  dedicated editor page not confirmed.
- **RTDB:** `sharedMarkers/{teamId}/{markerId}` (shared only). Favorites/markers sync
  writes under the user's cloud-sync payload.
- **Open:** whether Web has a personal-marker editor; the 4-category rollup model (iOS).

### Zone
- **iOS:** `Zone.swift`. Named geofence zones (distinct from redzones). **RTDB:**
  `zones/{id}`. **Open:** relationship to redzones and geofence monitoring inputs.

### Store
- **iOS:** `Store.swift` + `StoreOffers.swift`. Profile, deals, featured products,
  reviews, owner tools. **Android:** `stores/`. **Web:** `/stores`, `/store`,
  `/store/manage`. **RTDB:** `stores/{id}`, `storeDeals`, `storeProducts`,
  `storeReviews`. Image `Stores/{name}.png`.

### Guide
- **iOS:** `Guide.swift`, `GuideBookings.swift`, `GuideOffers.swift`. Profile, trips,
  deals, availability, bookings, announcements. **Android:** `guides/`, `bookings/`.
  **Web:** `/guides`, `/guide`, `/guide/manage`, `/bookings`. **RTDB:** `guides/{gid}`
  (indexed `ownerUserId`), `guideTrips`, `availability/{gid}`, `guideDeals`,
  `guideAnnouncements`, `guideBookings/{gid}/{bid}`, `userBookings/{uid}`. Image
  `Guides/{name}.png`.

### Regulation
- **iOS:** `Regulation.swift` + `RegulationDTO.swift`. State browser; PDF/link detail
  (`freshWaterReg`/`saltWaterReg`/`source`). Free for everyone. **Android:**
  `regulations/` + `bouquet` PDF viewer. **Web:** `/regulations` (read-only). **RTDB:**
  `regulations/{id}`, plus `states/{id}`, `counties/{id}`.

### Address
- **iOS:** `Address.swift`. Reused across tournaments/stores/guides. Holds
  `coordinates` + `type`. Forward-geocoded on create. **RTDB:** `address/{id}`.

### Tournament format
- **iOS:** `Format.swift`. Referenced by tournaments via `formatIds` (array).
  Cross-refs wiped+reinserted on edit. **RTDB:** `formats/{id}`.

### Coordinate
- **Shared facts:** lat/lng pairs embedded in redzones (polygon `latLngs`), addresses,
  markers, presence. Not a standalone RTDB node. **Open:** exact key names per context.

### Sponsor
- **iOS:** `Sponsor.swift` — exists but appears **unused** in tournament UI (INVENTORY
  open question). Resolves `Redzone.sponsorId` (header/image/name/description/website).
  **Android:** `Sponsor` model added in parity work. **RTDB:** `sponsors/{id}`.
  **Open:** intended usage.

### User
- **iOS:** `User.swift` (`@Model`) + `CurrentUser` (live auth/role state) + `PublicUser`
  mirror. Fields include `plan`, `appMode`, `auth` (role), `isGuide`/`isStoreOwner`/
  `isHostVerified`, `fcmToken`, `notificationPrefs`, `platform`.
- **Shared facts:** private `users/{uid}` (self/master) + world-readable `publicUsers/{uid}`
  (server-written). `platform ∈ {ios,android}` — Web is not a `platform` value.
- **Android:** `db/models` User + `profile/PublicUser.kt`. **Web:** `useAuth.tsx`,
  `publicUsers`. **RTDB:** `users/{uid}` (indexed `auth`), `publicUsers/{uid}`.
- **Open:** whether Web should write a `platform` value; full field list.

### Subscription plan
- **iOS:** `SubscriptionPlan` enum (`free` / `standard`→premium legacy / `premium`).
  Premium truth = StoreKit `Transaction.currentEntitlements`, mirrored **upgrade-only**
  to `users/{uid}/plan` and server `entitlements/{uid}`.
- **Android:** `subscription/SubscriptionPlan.kt`, `computeHasPremium` (Play-entitled OR
  `auth=='master'` OR `plan` premium/standard). **Web:** unlock via `entitlements` OR
  `plan==='premium'`. **RTDB:** `entitlements/{uid}` (server), `users/{uid}/plan`.
- See [`subscriptions.md`](subscriptions.md). **Open:** whether `plan` should ever be
  treated as a security boundary (Web notes it is UI-only).

## Additional shared concepts (not in the original list, but part of the domain)

Team, Fish (weigh-in entry), Series, RegisteredTeam, TripLog _(local-only)_, TrophyPost
(feed), Booking, Deal/Product, Review, RecordClaim (BAA), BankSpot, BoatTrack
_(local-only, Android/iOS)_, Presence. Each has an RTDB path in
[`realtime-database.md`](realtime-database.md) except the local-only ones.

## Open questions (domain-wide)

- Full per-field schemas and date conventions for every node (pending audit).
- Sponsor model intended usage.
- Whether Web participates in the `platform` field and presence.
- Personal-marker editor presence on Web.
- The "Member" vs "Bowfisher" fallback state on Android (parity conflict).
