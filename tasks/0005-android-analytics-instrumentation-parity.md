---
task_id: 0005-android-analytics-instrumentation-parity
parent_feature: (cross-cutting — analytics)
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: iOS analytics events across redzone / tournament / guide / store specs
status: proposed
deployment_authority: none
review_requirement: Michael approves the branch/PR
severity: MEDIUM
---

# 0005 — Android: Firebase Analytics instrumentation parity

## Scope

The audit found a **pervasive analytics gap**: Firebase Analytics events that iOS logs are
logged **nowhere** on Android across several groups, breaking cross-platform funnel parity.
Confirmed-missing events include (at least):

- **Redzone:** `redzone_viewed`, `redzone_entry`, `redzone_exit`
  (entry/exit also covered by task 0002).
- **Tournament:** `tournament_rsvp`, `tournament_comment`.
- **Guide:** guide `contact` / `review` events.
- **Store:** store `viewed` / `website` / `contact` / `review` events.

Work in `Android_Sector`:
- Add Firebase Analytics logging at the same interaction points iOS logs, using the **same
  event names and parameters** so the two platforms report into one funnel.
- Build the exact event list from the iOS source (grep iOS for `Analytics.logEvent` /
  the app's analytics wrapper) — do not guess names.

## Out of scope

- New analytics not present on iOS. This is parity, not expansion.
- iOS/Web changes.

## Contract references

- The relevant iOS specs (`redzone-map.md`, `tournament-profile.md`, `guide-profile.md`,
  `store-profile.md`) name the events; iOS source is the authoritative event list.

## Dependencies

- Firebase Analytics is available on Android (Firebase BoM already includes analytics).

## Acceptance criteria

- [ ] Event list derived from iOS source (names + params match).
- [ ] Each listed event fires at the equivalent Android interaction point.
- [ ] Verified in Firebase DebugView.

## Verification method

- On-device with DebugView enabled: exercise redzone view, RSVP, comment, guide contact,
  store view/website/contact → confirm each event appears with matching name/params.

## Completion record

_(empty until done)_
