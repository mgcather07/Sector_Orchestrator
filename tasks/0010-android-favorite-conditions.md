```yaml
task_id: 0010-android-favorite-conditions
parent_feature: favorite-conditions (candidate)
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: Sector/SwiftData/Models/FavoriteConditions.swift, MetricSheets.swift, DashboardShelves.swift — iOS #195
status: done
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

**2026-09-07 — implemented, in review.** Branch `feat/0010-favorite-conditions` off
`Michael-Master` → **PR [Android_Sector#22](https://github.com/mgcather07/Android_Sector/pull/22)**.
New store (57 lines) + 100 lines across 3 files, client-only:

- **`conditions/FavoriteConditions.kt`** (new) — `FavoriteConditionsStore`: device-local
  ordered list of starred metric keys (SharedPreferences), `StateFlow`, `toggle`/
  `isFavorite`; drops unknown keys on load. Init in `SectorApp`.
- **`conditions/ConditionsDetailScreen.kt`** — a star on each Conditions-section row
  (Moon/Wind/WaterTemp/Clarity/Pressure/Generation), only on rows the user can open;
  `conditions?sheet=<metric>` deep-link opens that metric's sheet; new public
  `FavoriteConditionsStrip` reuses the private `rowData` value source.
- **`home/DashboardTabBodies.kt`** — renders `FavoriteConditionsStrip` under a
  "Conditions" header on the Tonight tab; nothing when empty.

**Finding (corrects this task's "missing" premise):** the current Android dashboard has
**no** conditions-customization surface, BUT the old tile-strip machinery
(`ConditionsTilesStore` + `ShootingConditionsSection`, a reorder/hide grid) still exists
in source **parked with zero callers**. iOS deliberately moved *off* that tile-picker to
the star model (its `FavoriteConditions.swift` says so), so this implements the star
model and leaves the parked store untouched (removing it is a separate cleanup).

**Adaptation vs iOS:** iOS stars *inside* each metric's detail sheet; Android stars on the
metric's **row** in the Conditions list — because Android's per-metric sheets are
heterogeneous and premium-gated. Same metrics, same outcome. iPad tile grid (#198) is
out of scope / N-A on Android. Single value source (`rowData`) → a pinned row can't
disagree with its detail row.

**Verification:** `./gradlew compileDebugKotlin` clean (exit 0). **On-device pending** →
**`implemented_unverified`**: star Moon (free) → appears on Tonight tab; tap → sheet
opens; relaunch → persists; unstar → gone; Premium multi-star order + values match the
detail rows.

**Deployment impact:** none. **RTDB impact:** none.

**2026-09-07 — merged** to `Michael-Master` (squash `0e557af`, PR #22). `FavoriteConditions.kt`
present on trunk. Assessment stays **`implemented_unverified`** until the on-device check.
