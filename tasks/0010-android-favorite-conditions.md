```yaml
task_id: 0010-android-favorite-conditions
parent_feature: favorite-conditions (candidate)
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: Sector/SwiftData/Models/FavoriteConditions.swift, MetricSheets.swift, DashboardShelves.swift — iOS #195
status: approved
deployment_authority: none
review_requirement: Michael approves the branch/PR
```

# 0010 — Android favorite conditions

## Scope

On the **phone** layout: add a star to each weather-metric detail sheet that pins
that metric under the home Tonight "Conditions" section. Pinned metrics render in the
order the user starred them. Backed by a **per-device local preference** (iOS stores
comma-joined metric rawValues in order; Android: DataStore/SharedPreferences) that
survives launches and **drops unknown metric keys on load** so a future rename can't
leave a ghost pin. Render the pinned rows through the **same value source** the
conditions detail rows use (iOS `ConditionReadout`) so a metric can never show two
different numbers on two screens.

## Out of scope

- **iPad/large-screen star-driven tile grid + larger metric sheet (iOS #198) —
  EXCLUDED on Android.** This task is the phone favorite-conditions surface only.
- Any cloud sync of the favorite list — it is deliberately local/per-device (no RTDB).

## Contract references

- [`../parity/ios-delta-since-2026-08-23.md`](../parity/ios-delta-since-2026-08-23.md)
- Feature candidate **favorite-conditions** in
  [`../features/registry.md`](../features/registry.md)

## Dependencies

Shares the conditions value source with 0008/0011 — cleaner if the single-source
readout exists first, but not a hard blocker.

## Acceptance criteria

- Starring a metric on its sheet pins it under home Tonight "Conditions"; unstarring
  removes it; order is star order.
- Pins persist across launches and silently ignore unknown keys.
- A pinned metric's value matches the same metric's value on its detail sheet.
- No RTDB write; preference is device-local.

## Verification method

On-device: star/unstar several metrics, confirm home reflects it in order, relaunch to
confirm persistence, and confirm values match the detail sheet.

## Completion record

_(empty — approved, not yet implemented)_
