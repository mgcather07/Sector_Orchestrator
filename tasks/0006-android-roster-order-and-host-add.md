---
task_id: 0006-android-roster-order-and-host-add
parent_feature: tournament-registration / roster
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: iOS PR #191 — TournamentRosterView.swift, RegisterTeamView.swift (hostAdd mode)
status: proposed
deployment_authority: none
review_requirement: Michael approves the branch/PR
severity: MEDIUM
---

# 0006 — Android: roster registration-order numbering + sort, and host add-team

Parity with iOS PR #191, built to resolve a live tournament problem (nobody could tell who
registered first, and a team without a smart device couldn't be entered).

## Scope (two features)

### A. Roster numbered by registration order + sort filter
- Number each team by **registration order** (`registeredAt`), shown as a badge. The number is
  the team's **registration position** and stays fixed regardless of the display sort.
- Add a **sort filter**: *Registration order* (default) or *Alphabetical*. iOS previously
  hard-sorted A–Z with no numbers — same bug likely on Android; verify and fix.
- iOS reference: `TournamentRosterView.swift` — `orderedByRegistration`, `registrationRank`,
  `displayedTeams`, `sortMode`.

### B. Host add-team (for people without a smart device)
- The **host** (owner / co-host / master — mirror `Tournament.isHost(uid, isMaster)`) gets an
  **"Add team"** button on the roster that opens the registration form to enter a team by hand.
- **CRITICAL id rule:** self-registration uses a deterministic id (`"{tournamentId}_{uid}"`) so a
  user holds one team per tournament. A host adding **multiple** teams under their single uid
  MUST get a **unique id per team** (e.g. a UUID) or every add overwrites the last. This is the
  one thing that will silently corrupt the roster if missed.
- Host-add is **free-text** (no self-prefill), links to **no** season/series identity (the
  team isn't the host's squad — `seasonMatchKey` falls back to captain phone + name), sets
  `registeredByUserId = host uid` and `registeredAt = now`.
- Gate: host-only, and **hidden for externally-registered events** (BAA).
- iOS reference: `RegisterTeamView.swift` `hostAdd` flag + the roster's `addTeamButton`.

## Out of scope

- iOS/Web changes. Backend: **none** — `registeredTeams` already carries every field, and the
  RTDB rules already permit a host write (`registeredByUserId == auth.uid`). Do not touch rules.

## Contract references

- `tournament-registration` behavior; `RegisteredTeam` shape (`registeredAt`,
  `registeredByUserId`, `seriesTeamId`, …) — mirror iOS field names exactly.

## Dependencies

- None. Likely lives in `io/sector/co/tournament` (browse/registration/roster) — grep to confirm.

## Acceptance criteria

- [ ] Roster shows each team numbered by registration order; number stable across sorts.
- [ ] Sort toggles Registration order ↔ Alphabetical; default is Registration order.
- [ ] Host sees an "Add team" control; non-host does not; hidden for external events.
- [ ] Host can add **two** teams and BOTH persist (no overwrite) with a unique id each.
- [ ] Host-added team appears on the roster in correct registration order, written with the
      same `registeredTeams` field shape iOS writes.

## Verification method

- Device: open a tournament roster → confirm numbering + sort; as host, add two teams and
  confirm both persist and appear; confirm a non-host account has no Add button.

## Completion record

_(empty until done)_
