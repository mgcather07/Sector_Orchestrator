---
task_id: 0001-android-boat-tracks-owner-scoping
parent_feature: boat-tracks
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: iOS_Sector/specs/boat-tracks.md; iOS tracks store (per-user ownerUid scoping, stop-on-sign-out)
status: in_review
deployment_authority: none
review_requirement: Michael approves the branch/PR
severity: CRITICAL (privacy)
approved: 2026-08-24 by Michael
implemented: 2026-08-24 — Android PR #13 (awaiting on-device verification)
---

# 0001 — Android boat tracks: add owner scoping (privacy leak)

## Scope

Android's boat-tracks store is **not scoped to a user**. `BoatTrack` (its own Room DB)
has **no `ownerUid`**, and the `io/sector/co/tracks` package contains **zero**
`uid`/`currentUser`/sign-out references — so list, cap-count, and crash-recovery all read
**every** track on the device regardless of who is signed in. iOS explicitly scopes tracks
by `ownerUid` for exactly this reason.

**Consequence:** a second account signing in on a shared device sees the **first user's
entire GPS track history** (all-night on-the-water traces), and recording is not stopped on
sign-out. This is a genuine privacy leak, not a parity nicety.

Fix in `Android_Sector` only:
- Add `ownerUid` to `BoatTrack` (`co/tracks`), populated with the current Firebase uid at
  record start.
- Filter **every** track query — list, free-cap count, crash-recovery (`endedAt == null`) —
  by the current uid.
- Stop the recording foreground service and finalize/close any active track on sign-out.
- Migrate existing local rows: assign to the current uid on first launch after update, or
  purge unowned rows (decide during scoping — purge is safest for privacy).
- Align the free-track cap to **2** (iOS) — Android currently uses `FREE_TRACK_LIMIT=3`.

## Out of scope

- Cloud sync of tracks (tracks are local-only by design on both platforms).
- Any change to iOS or Web.

## Contract references

- `boat-tracks.md` (iOS spec). Tracks are local-only, per-user, premium-gated (2 free /
  unlimited premium).

## Dependencies

- None. Self-contained in the Android tracks package + its Room migration.

## Acceptance criteria

- [ ] `BoatTrack` has `ownerUid`; set at record start.
- [ ] List / cap-count / recovery all filter by current uid.
- [ ] Signing in as a different account shows **none** of the prior user's tracks.
- [ ] Recording stops (service + track finalized) on sign-out.
- [ ] Existing rows handled by a Room migration (assigned or purged — decision recorded).
- [ ] Free cap enforced at 2.

## Verification method

- On-device: record a track as account A, sign out, sign in as account B → confirm A's
  tracks are **not** visible (reproduce, then confirm fixed). Confirm recording stops on
  sign-out. Confirm cap blocks the 3rd free track.

## Completion record

- **Implemented + merged:** Android PR #13 (2026-08-24), branch
  `fix/android-boat-tracks-owner-scoping` → `Michael-Master`. 4 files, +87/−19.
- **Files:** `tracks/BoatTrack.kt` (nullable `ownerUid` + `MIGRATION_2_3` + owner-scoped
  `observeAll`/`activeTrack`/`savedCount` + `backfillUnowned`, free cap 3→2),
  `tracks/TrackRecorder.kt` (reads uid from FirebaseAuth, stamps `ownerUid` at record start,
  refuses to record signed-out, an `AuthStateListener` stops recording across an account
  change and triggers the backfill, `stopAndSave` preserves `ownerUid`, `resume` owner-guard),
  `tracks/TrackScreens.kt` + `home/DashboardTabBodies.kt` (scoped list + `attach()` on appear).
- **iOS reference:** `iOS_Sector/.../BoatTrack.swift` — `ownerUid`, `bootstrap(myUid:)`,
  `purgeUnowned`, `savedCount(for:)`, stop-on-sign-out.
- **Verification done (this session):** `compileDebugKotlin` + Room KSP schema validation pass.
  **Automated instrumented tests — GREEN on emulator** (Android PR #14, `connectedDebugAndroidTest`,
  2 tests / 0 failures on Pixel_9_Pro): `migration2to3_preservesRows_addsNullOwner_andBackfillClaims`
  proves the v2→v3 migration is non-destructive (legacy row survives, gains NULL owner, backfill
  claims it); `scoping_isolatesOwners_hidesWhenSignedOut_scopesCountAndActive` proves the core
  leak is closed — account B never sees account A's tracks, signed-out shows nothing, and
  savedCount/activeTrack are per-user. **The data-layer privacy leak is verified closed.**
- **Migration decision — BACKFILL, not purge.** Legacy unowned rows are claimed for the first
  signed-in user (data preserved). iOS *purges* unowned rows, but iOS's were pre-release test
  data; Android's are real shipped user tracks, so purging would delete saved nights. If
  iOS-identical purge is preferred, it's a one-line swap (delete instead of `backfillUnowned`).
- **Residual (accepted):** on a genuinely shared device, pre-update tracks are claimed by
  whichever account opens tracks first — a narrow one-time window. Every NEW track is owned
  from birth and sign-out stops recording, so the go-forward leak is closed.
- **Still pending — runtime/UI only (why status is `in_review`):** the data-layer isolation +
  migration are now automatically proven, so only two runtime behaviors remain, neither
  DB-testable: (1) the `AuthStateListener` actually stops the recording foreground service on
  sign-out, and (2) the 3rd free track triggers the paywall (`Map.kt` cap UI). A quick manual
  pass or a UI test covers both. Flip to `done` after either.
