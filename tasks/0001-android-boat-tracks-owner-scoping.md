---
task_id: 0001-android-boat-tracks-owner-scoping
parent_feature: boat-tracks
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: iOS_Sector/specs/boat-tracks.md; iOS tracks store (per-user ownerUid scoping, stop-on-sign-out)
status: proposed
deployment_authority: none
review_requirement: Michael approves the branch/PR
severity: CRITICAL (privacy)
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

_(empty until done)_
