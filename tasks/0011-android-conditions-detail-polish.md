```yaml
task_id: 0011-android-conditions-detail-polish
parent_feature: weather-metric-detail
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: Sector/SwiftData/Models/MetricSheets.swift, DashboardShelves.swift — iOS #193/#194/#195
status: done
deployment_authority: none
review_requirement: Michael approves the branch/PR
```

# 0011 — Android conditions detail polish

## Scope

Bring the Android conditions detail + dashboard to iOS #193/#194/#195 behavior:

1. **Wind chart (#195):** render sustained and gust **flush** (no gap/black between
   the two smoothed curves); drag-to-scrub shows **both** sustained and gust values;
   the scrub dot anchors to the last **observed** point and the **dashed forecast run
   begins at a synthetic "now" boundary** so solid = observed, dashed = forecast reads
   honestly.
2. **Fog-aware night banner (#193):** the tonight banner accounts for fog.
3. **Honest paywall CTA (#193):** premium CTA copy no longer overpromises.
4. **Cross-screen audit fixes (#194):** water-temp chart honors the range tabs
   (3wk/3mo/Season); graph range/tick fixes; copy de-duplication; color/band
   corrections so every tonight surface uses the same band scheme.

Adapt rendering to Compose Canvas / the Android charting in use — match behavior, not
Swift Charts APIs.

## Out of scope

- Engine and RTDB — unchanged.
- The favorite-conditions star (that's task 0010) and the score-cache (0008).

## Contract references

- [`../parity/ios-delta-since-2026-08-23.md`](../parity/ios-delta-since-2026-08-23.md)
- weather-metric-detail & dashboard rows, Group F of
  [`../parity/android-parity.md`](../parity/android-parity.md)

## Dependencies

None hard. Band-scheme correction should agree with whatever 0008 standardizes.

## Acceptance criteria

- Wind chart shows sustained+gust with no visual gap; scrub reports both values; solid
  vs dashed split sits at "now".
- Night banner reflects fog; paywall CTA copy matches iOS.
- Water-temp chart responds to the range tabs; graphs/colors/copy match the audited
  iOS result.

## Verification method

On-device: scrub the wind chart, toggle water-temp ranges, trigger a foggy-night state,
open the paywall, and eyeball band colors against iOS.

## Completion record

**2026-09-07 — implemented, in review.** Branch `feat/0011-conditions-band-scheme`
**stacked on 0010** → **PR [Android_Sector#23](https://github.com/mgcather07/Android_Sector/pull/23)**
(base is the 0010 branch; retarget to `Michael-Master` after #22 merges). **One line +
comment**, client-only.

**Key finding — most of 0011 was already at parity on Android** (verified in source, not
assumed). Only one applicable change remained:

- **Band scheme (the one fix):** the hourly score curve
  (`ConditionsDetailScreen.ScoreCurve`) drew its "Good" dashed reference at **60** (the
  retired 80/60/40 scheme). Moved to **65** to match the canonical Prime 80 / Good 65 /
  Fair 50 / Poor. `bandColor` is engine-rating-driven, so no other band hardcode existed.

Already present / N-A (no change):
- **Wind chart (#195):** `WeatherMetricDetail.TrendChart` already renders sustained +
  gust **flush** (gust is a full area *behind* the sustained line — no black gap by
  construction), injects a synthetic `now` sample so the solid past meets the dashed
  forecast at the Now marker, and the scrub readout already shows **both** sustained and
  gust. Ringed grabbable handle present. → already done.
- **Honest paywall CTA (#193):** already landed on Android via **merged PR #19**
  ("no free trial"). → already done.
- **Fog-aware night banner (#193):** Android has **no separate night banner**; fog
  awareness is already the "Fog likely tonight" insight card in the humidity detail. →
  N-A (no surface to make fog-aware without inventing one).
- **Water-temp range tabs (#194):** the water-temp sheet is a quick sheet with no
  3wk/3mo/Season tabs → the iOS tabs-not-filtering fix is **N-A**.

**Verification:** `./gradlew compileDebugKotlin` clean (exit 0). **On-device pending** →
**`implemented_unverified`**: confirm the hourly score curve's dashed "Good" line sits at
65.

**Deployment impact:** none. **RTDB impact:** none.

**2026-09-07 — merged** to `Michael-Master` (squash `3741878`, **PR #25** — the rebased
replacement for #23, which auto-closed when its stacked base branch was deleted). Trunk
score curve verified at `py(65)`. Assessment stays **`implemented_unverified`** until the
on-device check.
