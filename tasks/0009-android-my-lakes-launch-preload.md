```yaml
task_id: 0009-android-my-lakes-launch-preload
parent_feature: my-lakes
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: Sector/UI/Tab Bar/TabBar.swift + RootView.swift (LakeScorer.warmAll), MyLakes.swift — iOS #196
status: approved
deployment_authority: none
review_requirement: Michael approves the branch/PR
```

# 0009 — Android My Lakes preload at launch

## Scope

Warm the user's saved-lake scores at **app launch** (once the user/lake list is
available) instead of on first open of the My Lakes screen, so the screen opens
already populated. Critically, **decouple the score from the slow conditions
snapshot**: fetch the score via a direct engine `/conditions` call and render a
score-only card immediately; let the fuller snapshot (level, enrichment) fill in
afterward. iOS does this in `LakeScorer.warmAll()` triggered from the root view, with
a two-phase `request()` (phase 1 = engine score direct; phase 2 = snapshot enrich)
and a per-coordinate TTL so a relaunch inside the window doesn't re-fetch.

This pairs with task **0008** — the warm path should read through the same shared
cache/TTL, not a second independent one.

## Out of scope

- Engine and RTDB — unchanged.
- The My Lakes card visual design.

## Contract references

- [`../parity/ios-delta-since-2026-08-23.md`](../parity/ios-delta-since-2026-08-23.md)
- Group F / my-lakes note in [`../parity/android-parity.md`](../parity/android-parity.md)

## Dependencies

Best landed with or after **0008** (shares the per-coordinate cache/TTL). Not a hard
blocker.

## Acceptance criteria

- Saved-lake scores begin loading at launch, not on first My Lakes open.
- My Lakes opens showing scores already (no spinner-to-populate on entry in the common
  case).
- A score never blocks behind the slower snapshot; the snapshot enriches the card
  after the score shows.

## Verification method

On-device: cold-launch, wait briefly, open My Lakes → cards already scored. Confirm a
slow/blackholed snapshot host does not prevent the score from appearing.

## Completion record

_(empty — approved, not yet implemented)_
