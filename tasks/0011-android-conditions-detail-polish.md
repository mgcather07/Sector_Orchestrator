```yaml
task_id: 0011-android-conditions-detail-polish
parent_feature: weather-metric-detail
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: Sector/SwiftData/Models/MetricSheets.swift, DashboardShelves.swift — iOS #193/#194/#195
status: approved
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

_(empty — approved, not yet implemented)_
