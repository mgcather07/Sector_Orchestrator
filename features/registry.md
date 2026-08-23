# Feature Registry

Candidate features, extracted from current iOS behavior and the existing Features/Specs.
**These are candidates, not commitments.** Every candidate is `not_evaluated` until
Michael confirms its cross-platform scope; only then is a feature contract authored from
[`TEMPLATE.md`](TEMPLATE.md) and its platform matrix approved.

Legend — **Scope** columns use scope statuses; here all default to `not_evaluated`
(NE). The "Evidence" note summarizes what current inspection shows (assessment lives in
[`../parity/android-parity.md`](../parity/android-parity.md), not here).

> iOS scope reads `implemented` in reality for shipped features, but the orchestrator
> records it as **`not_evaluated`** until each contract is formally reviewed and
> approved — this keeps the registry honest that no cross-platform intent has been
> ratified yet. The recommended first contract to promote is **tournament-browsing**.

| # | Candidate feature | Source spec (iOS/Android) | iOS scope | Android scope | Web scope | Evidence note (not a commitment) |
|---|---|---|---|---|---|---|
| 1 | Tournament browsing | `tournament-browse.md` | NE | NE | NE | iOS shipped; Android present w/ drift (state filter/sort/50-mi historically dead code, reported wired post-audit); Web `/tournaments`. **Recommended pilot.** |
| 2 | Tournament detail | `tournament-profile.md` | NE | NE | NE | Present on all three. |
| 3 | Tournament filtering | `tournament-browse.md` §5-7 | NE | NE | NE | Android drift (see parity); Web filters present. |
| 4 | Tournament creation | `tournament-create.md` | NE | NE | NE | Premium/host. Android had gaps (owner picker, date+time) reported closed post-audit; Web `/tournaments/new`. |
| 5 | Redzone map display | `redzone-map.md` | NE | NE | NE | iOS MapKit; Android Google Maps; Web Mapbox GL. |
| 6 | Redzone-entry notifications | `geofence-notifications.md` | NE | NE | NE | iOS background region monitoring; **Android dormant (no bg-location perm)**; Web excluded by capability. |
| 7 | Personal map markers | `custom-markers.md` | NE | NE | NE | Local-only; iOS/Android full; Web unconfirmed editor. |
| 8 | Premium entitlements | `subscriptions.md` | NE | NE | NE | StoreKit / Play / Web-unlock. See contract. |
| 9 | New-tournament notifications | `notifications.md` | NE | NE | NE | Topic subscribe + create-write trigger. |
| 10 | Calendar integration | `tournament-profile.md` | NE | NE | NE | iOS EventKit; Android `ACTION_INSERT`. |
| 11 | Local tournament reminders | `notifications.md` | NE | NE | NE | 24h reminders iOS + Android. |
| 12 | Stores | `store-profile.md`, `store-management.md` | NE | NE | NE | Present all three. |
| 13 | Guides & booking | `guide-*.md` | NE | NE | NE | Present all three (large subsystem). |
| 14 | Regulations | `regulations.md` | NE | NE | NE | iOS/Android full; Web read-only. |

## Additional candidates observed (also `not_evaluated`)

From the ~40 iOS specs and current source, these are further candidates to promote when
their turn comes: tournament-comments, my-tournaments, tournament-live-results,
series-standings, teams, trips _(local-only)_, trophy-feed (+comments, new-post-badge),
follow-social, guide-deals/-trips/-announce-opening, search, location-browse, dashboard,
conditions-engine, tides, weather-metric-detail, open-meteo-fallback, baa-records-browser,
baa-record-submit, record-claims, auth, account-management, profile, app-modes,
subscriptions, cloud-sync, notifications, admin-tools, role-requests, navigation,
app-check, shared-markers, map-controls, boat-tracks _(Android/iOS only)_,
lake-conditions-alerts _(iOS-only spec)_.

**Android/iOS-only-so-far extras** (present in a platform, no iOS spec parity intent yet):
bank-spots, boat-tracks, my-lakes, dam-generation (Android). Record scope explicitly if
these are ever promoted.

## Promotion checklist (per feature)

1. Inspect current iOS behavior for the feature.
2. Reconcile its spec against current iOS + Android (+ Web) source.
3. Copy `TEMPLATE.md` → `features/<id>.md`; fill the platform matrix (scope +
   assessment, with rationales).
4. Michael reviews and approves the matrix.
5. Create a bounded task in [`../tasks/registry.md`](../tasks/registry.md) if
   implementation/verification is authorized.
6. Update this registry's scope columns to reflect the approved decision.
