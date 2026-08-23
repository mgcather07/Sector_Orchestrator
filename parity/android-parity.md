# Android Parity Assessment

**Date:** 2026-08-22 · **Method:** read-only inspection of current iOS + Android source,
reconciled against the existing spec set and Android's own parity docs. **No build or
runtime verification was performed** — so nothing here is `verified_implemented`.

## How to read this

- **Android scope status** reflects the **documented Android parity program** (Michael's
  `iOS-PARITY-ROADMAP.md` explicitly aims for feature parity). It is still subject to
  formal per-feature ratification via feature contracts. Deliberate divergences use
  `adapted`/`excluded` with a rationale.
- **Android assessment status** reflects current **evidence**. Because this is read-only,
  audit/spec "done" claims map to `implemented_unverified`; anything needing a device or
  build maps to `verification_pending`.
- **Confidence** — `high`/`medium`/`low` — is about how sure the *assessment* is, given
  read-only evidence.

**Staleness note:** Android's `PARITY-AUDIT.md` (2026-07-15) is ~5 weeks old and the
current source has moved past it (Chum Bucket, redzone submission now present). iOS
handoffs run to 2026-08-06. Where the audit and current source disagree, source wins.

## Backend-critical (verified directly by Android lead per audit)

| Item | Android scope | Assessment | Notes | Confidence |
|---|---|---|---|---|
| App Check (Play Integrity) | required | implemented_unverified | Installed before Firebase calls; console registration unverifiable read-only | high |
| Presence (`platform:"android"`) | required | implemented_unverified | `PresenceWriter` writes correct platform tag; reader is iOS master | high |

## Group A — Map & Spatial

| Feature | Existing spec | iOS evidence | Android evidence | Android scope | Assessment | Known gaps | Verification needed | Next action | Confidence |
|---|---|---|---|---|---|---|---|---|---|
| redzone-map | `redzone-map.md` | MapKit polygon/line/point, detail sheet, filters | `redzones/Map.kt` Google Maps overlays | adapted (Google Maps) | partial | No `redzone_viewed` analytics; distance subtitle; tournament-filter persistence; refresh-fail chip | On-device map render | Promote contract; decide analytics scope | high |
| custom-markers | `custom-markers.md` | Local-only; 4-category rollup; 25/∞ + 3/6 colors | `markers/` create/edit/list, guest markers | required | partial | No 4-category model; cap redirects at long-press vs save-time; no "Use current" in create | Device flow | Reconcile category model decision | high |
| map-controls | `map-controls.md` | Style/recenter/refresh/water-check | `redzones/Map.kt` | adapted | implemented_unverified | Style persisted (iOS resets); no refresh-fail chip | — | Accept drift or align | high |
| geofence-notifications | `geofence-notifications.md` | **Background** region monitoring | `GeofenceManager`/`ZoneManager` exist | required | **partial** | **Background geofencing dormant** — manifest lacks `ACCESS_BACKGROUND_LOCATION`, `refresh()` no-ops → foreground-only | **On-device; manifest audit** | **Bounded verification task** — see conflicts C1 | high |

## Group B — Tournaments (core)

| Feature | Existing spec | iOS evidence | Android evidence | Android scope | Assessment | Known gaps | Verification needed | Next action | Confidence |
|---|---|---|---|---|---|---|---|---|---|
| tournament-browse | `tournament-browse.md` | Filter/sort, 50-mi premium | `browse/BrowseListScreen.kt` | required | verification_pending | Audit found state filter/sort/50-mi as **dead code**; Phase-3 log says wired in — **needs confirmation in current source** | Read current source + device | **Recommended pilot** — promote contract, verify wiring | medium |
| tournament-profile | `tournament-profile.md` | RSVP/reminder/calendar/share | `tournament/TournamentProfile.kt` | required | implemented_unverified | LIVE color drift; no `tournament_rsvp` analytics | Device | Accept minor drift | high |
| tournament-create | `tournament-create.md` | Create/edit/delete, owner assign, formats | `features/AddTournament.kt` | required | verification_pending | Audit: no master Assign-Owner picker, date-only; Phase-3 log says fixed | Confirm in source + device | Verify owner picker + date+time | medium |
| tournament-comments | `tournament-comments.md` | Post/reply/like/report | `tournament/TournamentComments.kt` | required | implemented_unverified | Signed-out toast vs inline; no analytics | Device | — | high |
| my-tournaments | `my-tournaments.md` | Owned list, delete | `tournament/MyTournamentsScreen.kt` | required | implemented_unverified | No distinct not-logged-in/spinner states | Device | — | high |
| tournament-live-results | `tournament-live-results.md` | Register/check-in/weigh-in/results/side-pots/Chum | `tournament/live/*`, `results/` | required | partial | Audit missed Chum Bucket + "Bring $X" confirmation + pot badges; **`ChumBucket.kt`/`ChumDrawScreen.kt` now in source** | Device; confirm Chum wiring | Verify Chum + confirmation screen | medium |
| series-standings | `series-standings.md` | Flattened `series`, points engine | `tournament/series/*` | required | implemented_unverified | — | Device | — | high |

## Group C — Teams & Trips

| Feature | Existing spec | iOS evidence | Android evidence | Android scope | Assessment | Known gaps | Verification needed | Next action | Confidence |
|---|---|---|---|---|---|---|---|---|---|
| teams | `teams.md` | Two-sided teams/userTeams | `teams/TeamService.kt` | required | implemented_unverified | — | Device | — | high |
| trips | `trips.md` | **Local-only** logbook | `trips/` (local-only) | required | implemented_unverified | Audit found `restoreTrips()`/`pushTrips()` brushing local-only; Phase-log says removed | Confirm no cloud trip I/O in source | Verify local-only enforced (conflict C2) | medium |

## Group D — Guides & Booking

| Feature | Existing spec | iOS evidence | Android evidence | Android scope | Assessment | Known gaps | Verification needed | Next action | Confidence |
|---|---|---|---|---|---|---|---|---|---|
| guide-profile | `guide-profile.md` | Book gate on `hasTrips` | `guides/GuideProfile.kt` | required | partial→implemented_unverified | Audit: gated on availability not `hasTrips`; Phase-log says fixed | Confirm in source | Verify gate | medium |
| guide-trips | `guide-trips.md` | Duration removed 2026-07-05 | `guides/TripEditScreen.kt` | required | partial→implemented_unverified | Audit: Duration field lingered; Phase-log says removed | Confirm in source | Verify field removed | medium |
| guide-deals | `guide-deals.md` | Promo deals, featured rules | `guides/ManageDealsScreen.kt` | required | partial | Manage sort order; editor constraints | Device | — | medium |
| guide-booking-availability | `guide-booking-availability.md` | Slots, 30-min, past-block | `guides/ManageAvailabilityScreen.kt` | required | implemented_unverified | Minor mark distinction | Device | — | high |
| guide-booking-customer | `guide-booking-customer.md` | Browse/request/my-requests | `guides/BookTripScreen.kt` | required | implemented_unverified | Sign-in guard Toast vs dialog | Device | — | high |
| guide-booking-owner | `guide-booking-owner.md` | Accept (atomic seats)/decline | `guides/GuideBookingsScreen.kt` | required | implemented_unverified | — | Device | — | high |
| guide-announce-opening | `guide-announce-opening.md` | Followers push | `guides/BookingService.kt` | required | implemented_unverified | — | Device | — | high |

## Group E — Stores, Regulations, Search, Browse

| Feature | Existing spec | iOS evidence | Android evidence | Android scope | Assessment | Known gaps | Verification needed | Next action | Confidence |
|---|---|---|---|---|---|---|---|---|---|
| store-profile | `store-profile.md` | Profile/deals/products/reviews | `stores/StoreProfile.kt` | required | implemented_unverified | **No Firebase Analytics events** | Device | Decide analytics scope | high |
| store-management | `store-management.md` | Create/edit, deals, products | `stores/AddStore.kt` | required | implemented_unverified | Website validation weak; product editor requires price | Device | — | high |
| regulations | `regulations.md` | PDF/link, `gs://` via Storage | `regulations/` + bouquet | required | partial→implemented_unverified | Audit: no Storage-SDK branch (`gs://` fail), sort by abbr, no image fallback; Phase-log says fixed | Confirm in source + device | Verify `gs://` branch | medium |
| search | `search.md` | People + local, no filters | `search/` | required | implemented_unverified | Empty-placeholder copy | Device | — | high |
| location-browse | `location-browse.md` | Shared location sheet + city search | `browse/BrowseListScreen.kt`, `filter/` | adapted | partial | City-search sheet unwired dead code; "Nearby" chip = device-location only | Device | Decide adapt vs align | medium |

## Group F — Dashboard & Conditions

| Feature | Existing spec | iOS evidence | Android evidence | Android scope | Assessment | Known gaps | Verification needed | Next action | Confidence |
|---|---|---|---|---|---|---|---|---|---|
| dashboard | `dashboard.md` | Gauge, tiles, banners | `home/Home.kt`, `conditions/` | required | partial | Compact water row ignored picked-city (Phase-log says fixed); master tools omit some admin panels | Confirm in source | Verify water-row override | medium |
| conditions-engine | `conditions-engine.md` | v2 gates/seeability/regime; server-fed by Sector_Engine | `conditions/BowfishingConditions.kt` | adapted | **intentionally_different** | Deliberately simplified; no hard gates/seeability/regime/tailwater/confidence-fade; **scores may diverge from iOS on identical inputs** | Cross-check vs Sector_Engine | Decide: adopt engine service or accept divergence | medium |
| weather-metric-detail | `weather-metric-detail.md` | 5 metric sheets, Canvas moon | `conditions/WeatherMetricDetail.kt` | required | partial→implemented_unverified | Audit: emoji moon vs Canvas; Phase-log says Canvas added | Confirm in source | — | medium |
| tides | `tides.md` | NOAA nearest station | `conditions/NoaaTideService.kt` | required | implemented_unverified | — | Device | — | high |
| open-meteo-fallback | `open-meteo-fallback.md` | Ordered host failover | `conditions/WeatherService.kt` | required | implemented_unverified | — | — | — | high |
| lake-conditions-alerts | `lake-conditions-alerts.md` (**iOS-only spec**) | iOS lake alerts | Not confirmed on Android | not_evaluated | unknown | iOS-only spec (absent from Android spec mirror) | Decide scope | Ask Michael if Android-bound | low |

## Group G — Feed & BAA

| Feature | Existing spec | iOS evidence | Android evidence | Android scope | Assessment | Known gaps | Verification needed | Next action | Confidence |
|---|---|---|---|---|---|---|---|---|---|
| trophy-feed | `trophy-feed.md` | Paged feed, compose, spot-privacy | `feed/FeedScreen.kt` | required | partial→implemented_unverified | Audit: Share button missing; Phase-log says added | Confirm in source | Verify Share action | medium |
| trophy-feed-comments | `trophy-feed-comments.md` | Comment thread | `feed/PostDetailScreen.kt` | required | implemented_unverified | — | Device | — | high |
| feed-new-post-badge | `feed-new-post-badge.md` | Numbered badge | `feed/FeedBadge.kt` | required | implemented_unverified | — | Device | — | high |
| baa-records-browser | `baa-records-browser.md` | Browse by category (separate BAA project, REST) | `records/BAARecordsService.kt` | required | implemented_unverified | Evolved past stale spec (matches newer iOS) | Device | — | high |
| baa-record-submit | `baa-record-submit.md` | Submit via REST, `source` tag | `records/SubmitRecordScreen.kt` | required | partial→implemented_unverified | Audit: close-up/mouth picker missing, cap 3→5, "Still needed" list; Phase-log says fixed | Confirm in source | Verify photo picker | medium |
| record-claims | `record-claims.md` | Claim state machine + mirror | `records/RecordClaimService.kt` | required | partial→implemented_unverified | Audit: master backfill (7e) + re-read guard missing; Phase-log says fixed | Confirm in source | Verify backfill | medium |

## Group H — Auth, Profile, Social & Platform

| Feature | Existing spec | iOS evidence | Android evidence | Android scope | Assessment | Known gaps | Verification needed | Next action | Confidence |
|---|---|---|---|---|---|---|---|---|---|
| auth | `auth.md` | Email/Google/Apple/guest | `login/` | required | implemented_unverified | **Apple = excluded (N/A on Android)**; forgot-pw pre-check (Phase-log fixed) | Device | — | high |
| account-management | `account-management.md` | Logout/delete ordering | `profile/DeleteAccountScreen.kt` | required | implemented_unverified | — | Device | — | high |
| profile | `profile.md` | Avatar/cover, public mirror | `profile/UserProfile.kt` | required | implemented_unverified | — | Device | — | high |
| follow-social | `follow-social.md` | Two-edge follow | `profile/FollowService.kt` | required | implemented_unverified | Counts one-shot not live | Device | — | high |
| app-modes | `app-modes.md` | Angler/Booker onboarding | `profile/AppMode*` | required | implemented_unverified | — | Device | — | high |
| subscriptions | `subscriptions.md` | StoreKit → Play on Android | `subscription/*` | required | verification_pending | **Needs live Play Console product + device purchase/restore** | **On-device purchase** | Bounded device-verification task | high |
| cloud-sync | `cloud-sync.md` | Favorites/markers sync; trips local-only | `sync/UserDataSyncService.kt` | required | partial | Trips local-only concern (conflict C2) | Confirm in source | Verify trip I/O removed | medium |
| notifications | `notifications.md` | FCM, topics, reminders | `notifications/*` | required | partial | Deep-link tap routing not fully traced; geofence dormant (C1) | Device | — | medium |
| cloud-functions (triggers) | `cloud-functions.md` | ~30 functions (backend repo) | Trigger-writes present | required | implemented_unverified | `notifyNewRedzone`: audit said no draft write; **`RedzoneDraftService` now in source** | Confirm redzone write | Verify redzone trigger | medium |
| admin-tools | `admin-tools.md` | Approvals, redzones tab, env policy | `roles/`, `admin/` | required | partial | Audit: no Redzones tab, no live role sync, no force-prod guard; **source now has redzone submission + live role listener** | Confirm in source + device | Verify admin approvals | medium |
| navigation | `navigation.md` | Angler/Booker tabs, deep links | `menu/`, `navigation/` | required | implemented_unverified | — | Device | — | high |
| app-check | `app-check.md` | Play Integrity | `AppCheck.kt` | required | implemented_unverified | Console registration unverifiable | Console | — | high |

## Deliberate divergences (NOT gaps)

| Item | Type | Rationale |
|---|---|---|
| Apple Sign-In absent on Android | excluded | No Apple provider on Android by design |
| Google Maps (not Mapbox) | adapted | Explicit decision in `iOS-PARITY-ROADMAP.md` |
| Conditions engine simplified | adapted / intentionally_different | Android reimplements; scores may diverge; server-fed by shared Sector_Engine |
| Trips local-only | required-but-local | No cloud sync/photo upload by product rule |
| iPad/large-screen layouts | excluded | iOS-only, out of Android scope |

## What still needs Michael's decision

1. **Ratify Android scope per feature** (this table reads scope from the documented
   parity program; formal approval turns candidates into contracts).
2. **Background geofencing** (conflict C1): declare `ACCESS_BACKGROUND_LOCATION` + verify,
   or accept foreground-only?
3. **Conditions-engine divergence**: adopt the shared Sector_Engine numbers, or accept
   Android's simplified scoring?
4. **Analytics scope**: are Firebase Analytics events (present on iOS spec) required on
   Android?
5. **"Member" vs "Bowfisher" no-name fallback** (conflict C3): update Android.

## Recommended first promotion

**tournament-browsing** — it exercises shared models and platform presentation without
background location, subscriptions, or other high-risk capabilities, and it has a
concrete, verifiable open question (was the state/sort/50-mi filter wiring actually
completed, or is it still dead code?). See
[`../docs/workflow.md`](../docs/workflow.md) for the promotion workflow.
