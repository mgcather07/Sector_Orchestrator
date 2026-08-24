---
task_id: 0002-android-background-geofencing
parent_feature: geofence-notifications
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: iOS_Sector/specs/geofence-notifications.md; iOS background CLRegion monitoring
status: proposed
deployment_authority: none
review_requirement: Michael approves the branch/PR (background-location has Play policy + privacy implications)
severity: HIGH
---

# 0002 — Android background geofencing (dormant → live)

## Scope

The full Android geofencing architecture exists — `GeofenceManager.kt` (nearest-20 /
4mi + cache), `ZoneManager.kt` (foreground polygon refinement), `GeofenceBroadcastReceiver.kt`
(master + per-type gating), `ZoneOccupancy`, `BootReceiver` — but it is **dormant**:
`AndroidManifest.xml` does not declare `ACCESS_BACKGROUND_LOCATION` and it is never
requested, so `GeofenceManager.refresh()` / `rearmFromCache()` no-op by design
(`GeofenceManager.kt:68-72`). Result: zone-entry alerts fire **foreground-only**, defeating
the feature's "alerts you when the app is closed" purpose. (Same class as iOS TASK-031.)

This tracks orchestrator conflict **C1** — it needs Michael's decision before code, because
background location requires a Play Store policy justification.

Work in `Android_Sector`:
- Declare `ACCESS_BACKGROUND_LOCATION`; add a clear rationale UX to request it (separate
  from foreground location, per Android 11+ flow).
- Wire `refresh()` / `rearmFromCache()` to actually arm geofences once granted.
- Prepare the Play Console background-location declaration + justification.
- Add the missing `redzone_entry` / `redzone_exit` analytics events (parity with iOS).

## Out of scope

- iOS/Web changes. The redzone data model and gating logic (already built) are unchanged.

## Contract references

- `geofence-notifications.md`. Nearest-20-within-4mi, per-type gating, circular approx +
  polygon refinement, entry/exit analytics.

## Dependencies

- **Michael's decision** on shipping background location (Play policy). Blocked until then.

## Acceptance criteria

- [ ] `ACCESS_BACKGROUND_LOCATION` declared + requested with rationale.
- [ ] Geofences arm in the background; a zone entry with the app closed fires an alert.
- [ ] Per-type + master gating still respected in the background path.
- [ ] Play Console justification prepared.
- [ ] `redzone_entry` / `redzone_exit` analytics logged.

## Verification method

- On-device: grant background location, close the app, cross a monitored zone boundary →
  confirm the entry notification fires. Confirm gating (disabled type = no alert).

## Completion record

_(empty until done)_
