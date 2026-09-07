```yaml
task_id: 0008-android-score-consistency
parent_feature: conditions-engine
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: Sector/SwiftData/Models/EngineAPIClient.swift (ConditionsMemo), MyLakes.swift, UI/Map/Details.swift — iOS #199/#200/#194
status: approved
deployment_authority: none
review_requirement: Michael approves the branch/PR
```

# 0008 — Android cross-surface score consistency ("Cory bug")

## Scope

Make every tonight-score surface on Android return the **same** score for the same
coordinate within a short window, the way iOS does after PR #199/#200. Introduce a
single process-wide in-memory cache keyed by a **rounded** coordinate (iOS uses
`%.3f,%.3f` ≈ 110 m) storing the raw engine response, with a **10-minute TTL**
(`ConditionsMemo.ttl = 10*60` on iOS). The engine client checks the cache before the
network and stores every successful response into it, so all callers for the same
~coordinate inside the window decode identical bytes → identical score → identical
rounding. Give the My Lakes scorer and the map-pin/redzone conditions path the same
TTL guard (a `loadedAt` timestamp compared against the shared TTL) so they read
through the cache instead of re-fetching on their own cadence. A single
`invalidate()` clears all of them together on explicit refresh / account switch.

Surfaces to route through the cache (enumerate and confirm in Android source):
dashboard compact score ring, any score/conditions detail screen, the redzone/lake
**map pin**, **My Lakes** cards, and **Where-to-go**.

## Out of scope

- The Cloud Run Sector_Engine and any RTDB path/field/rule/Function — **unchanged**.
  Both platforms already score off the one shared engine; the divergence was purely
  local caching/rounding. Do not "fix" scores server-side here.
- Any visual redesign of the score surfaces.

## Contract references

- [`../parity/ios-delta-since-2026-08-23.md`](../parity/ios-delta-since-2026-08-23.md)
  (detail section on this fix)
- Feature candidate: conditions-engine (Group F of
  [`../parity/android-parity.md`](../parity/android-parity.md) — already notes scores
  are structurally shared; this closes the *client-side* divergence that claim predates)

## Dependencies

None. Client-only, zero RTDB.

## Acceptance criteria

- One shared, coordinate-rounded, TTL-bounded cache exists and every score surface
  reads through it.
- Two different surfaces opened on the same coordinate within 10 minutes show the
  **same** number.
- Two devices on the same coordinate agree within the window (engine determinism
  permitting).
- Explicit refresh / sign-out clears the cache for all surfaces at once.

## Verification method

On-device: open the same lake/coordinate on ≥2 surfaces and confirm identical scores;
repeat across two devices; confirm refresh invalidates. Read-only review yields
`implemented_unverified` until this device pass runs.

## Completion record

_(empty — approved, not yet implemented)_
