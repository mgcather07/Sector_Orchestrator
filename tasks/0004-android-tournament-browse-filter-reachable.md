---
task_id: 0004-android-tournament-browse-filter-reachable
parent_feature: tournament-browse
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: iOS_Sector/specs/tournament-browse.md (filter sheet: When/Sort/State/50-mi)
status: in_review
deployment_authority: none
review_requirement: Michael approves the branch/PR
severity: MEDIUM (dead functionality, easy fix)
approved: 2026-08-24 by Michael
implemented: 2026-08-24 — Android PR #17 (compiles; quick on-device tap-through pending)
---

# 0004 — Android tournament browse: make the filter sheet reachable

## Scope

Android's `BrowseListScreen.kt` has a **fully built** `TournamentFilterSheet` (When / Sort /
State / 50-mi premium), but **nothing opens it** — the toolbar filter button was removed and
nothing sets `showFilterSheet = true`. So **State filtering** and **Latest/Name sort** are
dead functionality: only Soonest / Past-desc are reachable. (This corrects the prior doc's
stale "filter is dead code" claim — the sheet works; it's the *entry point* that's missing.)

Work in `Android_Sector`:
- Restore a filter entry point (toolbar filter button) that sets `showFilterSheet = true`.
- Add an "Adjust filters" button to the empty state (iOS parity).
- Confirm State filter + Latest/Name sort actually drive the list once reachable.

## Out of scope

- Any change to the filter logic itself (it exists) beyond wiring the entry point.
- Map trophy-pin clustering (tracked separately if needed).

## Contract references

- `tournament-browse.md` §5-7 (filter sheet, sort, state, 50-mi premium radius).

## Dependencies

- None.

## Acceptance criteria

- [ ] A visible control opens the existing filter sheet.
- [ ] Applying a State filter narrows the list; Latest/Name sort reorders it.
- [ ] Empty state offers "Adjust filters".

## Verification method

- On-device: open browse, tap filter, set State + sort, confirm the list responds.

## Completion record

- **Root cause confirmed:** `browse/BrowseListScreen.kt` — the `TournamentFilterSheet`
  (State / Sort / Nearby) and its `showFilterSheet` state were fully built, but the toolbar
  filter icon had been removed (`actions = { … Filter icon removed … }`) and **nothing set
  `showFilterSheet = true`**. So State + Latest/Name sort were reachable by no path.
- **Fix (Android PR #17):** restored a `FilterList` `IconButton` in the browse `TopAppBar`
  that opens the sheet, tinted primary when `anyFilterActive`; added an **"Adjust filters"**
  `TextButton` to the empty state when filters are active.
- **Verification done:** `compileDebugKotlin` passes.
- **Still pending — quick on-device tap-through (why `in_review`):** open Tournaments → tap
  the filter icon → sheet opens → set a State + Latest/Name sort → list responds; confirm the
  icon tints when a filter is active and "Adjust filters" shows on an empty filtered result.
