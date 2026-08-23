# Contract: Notifications

**Status:** Draft. Based on read-only inspection on 2026-08-22.

Sector notifications come in three families: **push** (FCM, often triggered by Cloud
Functions on RTDB writes), **local** (scheduled reminders), and **geofence** (redzone
entry alerts). Firebase Cloud Messaging is confirmed on iOS and Android.

## Current iOS behavior (reference)

- **FCM:** APNs→FCM token handoff written to `users/{uid}/fcmToken` (`AppDelegate.swift`).
- **Push topics:** state-based tournament topics `tournaments_{STATE}` / `tournaments_all`
  (`NotificationManager.swift`), reconciled on home-state change.
- **Local notifications** (`UNUserNotificationCenter`): tournament **24h reminders**
  (`UNTimeIntervalNotificationTrigger`), calendar-style (`UNCalendarNotificationTrigger`),
  geofence-entry alerts. Tapping deep-links to the tournament profile.
- **Geofencing** (`ZoneManager.swift`, `Zone+Geofencing.swift`): **background region
  monitoring** via `CLLocationManager` + `CLCircularRegion` (`startMonitoring`,
  `didEnterRegion`/`didExitRegion`/`didDetermineState`, significant-location-change).
  Monitors ~20 nearest redzones within ~4 mi, survives suspension via a
  `monitored_zones.json` cache, refines with foreground ray-cast polygon containment,
  per-zone-type toggles, and While-Using→Always escalation.

## Server-side triggers (Cloud Functions)

The deployed functions (iOS `functions/`, Web `functions/`) send most pushes off RTDB
writes — so a client "sends" a notification simply by writing the right path:

| Function | Trigger / purpose |
|---|---|
| `notifyNewTournament` | new tournament → FCM state topic |
| `notifyNewSubscriber` | `users/{uid}/plan`→premium → master |
| `notifySupportMessage` | support thread message → master or user |
| `notifyNewRoleRequest` | role request pending → masters |
| `notifyNewRedzone` / `notifyRedzoneSubmission*` | redzone draft/submission → masters |
| `notifyGuideOpening` | guide announces slot → that guide's followers |
| `notifyBooking` | booking mirror + seat release + push guide/customer |
| `notifyTeamInvite` | team invite push |
| `activity*` (onPostLike/onComment/onFollow/…) | write `activity/{uid}` feed entries |
| author-stamp functions | server-stamp trophy/comment authors (anti-impersonation) |

## Confirmed shared behavior

- FCM token written to `users/{uid}/fcmToken` (iOS + Android).
- Tournament state topics `tournaments_{STATE}` / `tournaments_all` (iOS + Android).
- Local 24h tournament reminders with boot re-arm (iOS + Android).
- Notifications are largely **RTDB-write-triggered** — no new server code is needed for a
  platform to fire an existing notification; it writes the same path.

## Platform differences

| Capability | iOS | Android | Web |
|---|---|---|---|
| FCM push | ✅ | ✅ (`SectorMessagingService`) | (not a push target; browser) |
| State tournament topics | ✅ | ✅ (`TournamentTopics`) | n/a |
| Local 24h reminders | ✅ (`UN*Trigger`) | ✅ (`AlarmManager` + `BootReceiver`) | n/a |
| **Background geofence zone-entry** | ✅ (region monitoring) | **⚠️ dormant** — code exists, but manifest lacks `ACCESS_BACKGROUND_LOCATION`, so it no-ops; alerts are **foreground-only** | **excluded** (background geofencing is not reliable on the Web) |
| Per-zone-type toggles | ✅ | ✅ (`NotificationPrefs`) | n/a |

- **Android background geofencing is the key flag.** The roadmap claims it complete, but
  the manifest reality makes it dormant. This is recorded in
  [`../parity/unresolved-conflicts.md`](../parity/unresolved-conflicts.md) and is a strong
  candidate for a bounded verification task.
- **Web excludes background geofencing** by platform capability — record as `excluded`
  with rationale in the `geofence-notifications` feature contract.

## Assumptions (to verify)

- Android deep-link-from-tap routing is wired end-to-end (audit noted it as not fully
  traced).
- Whether all `activity/*` and author-stamp functions have dev+prod variants matching
  each RTDB instance.

## Change-control expectations

- New notification types usually mean a new RTDB write path or a new Cloud Function —
  both are gated changes (contract + rules + deployment approval).
- Changing topic naming (`tournaments_{STATE}`) affects all subscribers and the sending
  function simultaneously — treat as a coordinated migration.

## Open questions

- Should Android declare `ACCESS_BACKGROUND_LOCATION` and enable the geofence path (needs
  Play Store background-location justification + on-device verification)?
- Are Storage/Cloud Messaging/Functions usage identical across dev and prod instances?
  ([`../docs/open-questions.md`](../docs/open-questions.md) Q10.)
