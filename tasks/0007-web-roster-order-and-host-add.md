---
task_id: 0007-web-roster-order-and-host-add
parent_feature: tournament-registration / roster
authorized_repositories:
  - Web_Sector
platform: web
ios_behavior_reference: iOS PR #191 — TournamentRosterView.swift, RegisterTeamView.swift (hostAdd mode)
status: in_review
deployment_authority: none
review_requirement: Michael approves the branch/PR
severity: MEDIUM
approved: 2026-08-24 by Michael
implemented: 2026-08-24 — Web main a3e08d6 (next build passes; manual owner check pending)
---

# 0007 — Web: roster registration-order numbering + sort, and host add-team

Parity with iOS PR #191. Web is the multi-role app (customer / owner / admin), so host-add
belongs to the **owner/host** surface.

## Scope (two features)

### A. Roster numbered by registration order + sort filter
- Number each team by **registration order** (`registeredAt`), shown next to the name. The
  number is the registration **position** and is stable regardless of the display sort.
- Add a **sort control**: *Registration order* (default) or *Alphabetical*.
- iOS reference: `TournamentRosterView.swift` — `orderedByRegistration`, `registrationRank`,
  `displayedTeams`, `sortMode`.

### B. Host add-team (for people without a smart device)
- The **host/owner** (mirror `Tournament.isHost` — owner / co-host / master) gets an
  **"Add team"** action on the roster to enter a team by hand (team name, captain phone,
  1–4 shooters, side pots).
- **CRITICAL id rule:** self-registration uses a deterministic id (`"{tournamentId}_{uid}"`)
  = one team per user. A host adding **multiple** teams under one uid MUST assign a **unique
  id per team** (UUID) or each add overwrites the last. Do not miss this.
- Host-add is free-text, links to **no** season/series identity (`seasonMatchKey` falls back to
  captain phone + name), sets `registeredByUserId = host uid` and `registeredAt = now`.
- Hidden for externally-registered events (BAA).
- iOS reference: `RegisterTeamView.swift` `hostAdd` flag + the roster's `addTeamButton`.

## Out of scope

- iOS/Android changes. Backend: **none** — `registeredTeams` already carries every field, and the
  (iOS-owned, Web-mirrored) RTDB rules already permit a host write. Do not edit
  `database.rules.json` here (it is a mirror — orchestrator ADR 0002).

## Contract references

- `tournament-registration`; `RegisteredTeam` field shape — mirror iOS field names exactly.

## Dependencies

- None. Registration likely lives in `src/components/tournament/Registration.tsx` and the
  tournament page(s) under `src/app/tournament*` — confirm before editing.

## Acceptance criteria

- [ ] Roster shows each team numbered by registration order; number stable across sorts.
- [ ] Sort toggles Registration order ↔ Alphabetical; default Registration order.
- [ ] Host/owner sees an "Add team" action; a non-owner customer does not; hidden for external.
- [ ] Host can add **two** teams and BOTH persist (unique id each — no overwrite).
- [ ] Written with the same `registeredTeams` field shape iOS writes.

## Verification method

- `next build` passes; manual: open a tournament roster → numbering + sort work; as owner, add
  two teams and confirm both persist; confirm a non-owner has no Add action.

## Completion record

- **Implemented + pushed:** Web `main` a3e08d6 (2026-08-24), 1 file
  (`src/components/tournament/Registration.tsx`).
- **iOS reference:** PR #191; **Android:** PR #16.
- **Roster:** the flattened per-shooter list is numbered by the **team's registration
  position** (once per team, on its first shooter row) instead of a raw row ordinal; a
  **sort control** (Registration order default / Alphabetical). `subscribeRegisteredTeams`
  already returns teams registration-sorted, so registration order was already the base.
- **Host add-team:** host/owner/master gets a `+ Add team` button that opens the register
  modal in host-add mode (free-text, always a new team, "Add a team" copy).
- **No unique-id guard needed:** Web already IDs new teams with `newId()` (a push id), so
  the deterministic-`{tid}_{uid}` overwrite trap that iOS/Android had does not exist here —
  every host-add is naturally unique. registeredByUserId = host uid; registeredAt = now.
- **Verification done:** `next build` passes (compile + lint + type-check). No backend change.
- **Still pending — manual (why `in_review`):** as owner, open a tournament roster → confirm
  numbering + sort; add TWO teams and confirm both appear (unique ids); confirm a non-owner
  has no Add button; confirm the per-team number reads sensibly on multi-shooter teams.
