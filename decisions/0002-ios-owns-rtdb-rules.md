# 0002 — iOS owns the Realtime Database rules

- **Status:** accepted
- **Date:** 2026-08-23
- **Deciders:** Michael (with Claude Code)

## Context

The shared RTDB security rules (`database.rules.json`) were checked into **two** repos —
iOS and Web — and **both deploy them**. The two copies had drifted (conflict
[C4](../parity/unresolved-conflicts.md)), so whichever repo deployed last silently
overwrote the other's rules in the live database. A precise diff of the two files on
2026-08-23 found the drift was smaller than C4 first described — the `entitlements` node
was actually identical — but real:

- Web had a top-level **`playAccountTokens`** node (Google Play purchase-token map) that
  iOS lacked.
- iOS had a top-level **`offerCodes`** node (the cross-platform redeem-QR pool) that Web
  lacked.
- **`redzoneDrafts`** `.write`: Web also allowed `users/{uid}/plan === 'premium'` writers;
  iOS only checked the older `entitlements/{uid}/active` flag.
- **`registeredTeams`** `.indexOn`: Web also indexed `seriesTeamId`.

Deploying either copy as-is would delete a node the other platform depends on — a
standing production risk that also blocked deploying the redeem-QR rules from iOS.

## Decision

**The iOS repository (`iOS_Sector/database.rules.json`) is the single canonical source of
truth for RTDB rules.** The file was reconciled into a **strict superset** that contains
everything both platforms need (iOS PR #189):

- added `playAccountTokens`,
- widened `redzoneDrafts` `.write` to accept `plan === 'premium'`,
- added `seriesTeamId` to `registeredTeams` `.indexOn`,
- kept `offerCodes`.

Verified: iOS is now a strict superset of Web (0 Web nodes missing, 0 shared nodes
differing). Going forward:

1. Rules are **edited only in the iOS repo.**
2. The Web repo's `database.rules.json` is kept as an **exact mirror** of iOS's (or Web is
   removed from rules deployment) so a Web deploy can never reintroduce drift.
3. Rules are **deployed once, from iOS** (`firebase deploy --only database`), as a
   deliberate, approval-gated step — never automatically.

Scope note: this ADR decides **rules** ownership. **Cloud Functions** ownership is a
separate, still-open question — iOS `functions/` and Web `functions/` currently hold
*different* function sets (iOS: notifications + `redeem`; Web: appstore/play
notifications, the conditions proxy, admin utilities). That reconciliation is tracked in
open question Q19.

## Alternatives

- **Web owns rules.** Rejected — the app's behavioral source of truth and the primary
  developer's repo is iOS; iOS also holds the deployed `functions/` the rules pair with.
- **Keep dual ownership, coordinate by hand.** Rejected — the drift already reached
  production silently once; manual coordination is exactly what failed.
- **A third neutral repo owns rules.** Rejected as over-engineering for a solo developer;
  adds a hop without removing the mirror requirement.

## Consequences

- An iOS rules deploy is now safe — it cannot delete a Web/Android-needed node.
- The redeem-QR feature (iOS PR #187) can deploy its `offerCodes` rules from iOS without
  clobbering Play token support.
- One clear mental model: edit in iOS, mirror to Web, deploy from iOS.

## Risks

- **Web can still deploy rules** until its copy is made a mirror (or removed from its
  deploy). Until then, a Web deploy would delete `offerCodes`. Closing this is the
  remaining task.
- If a future Web/Android need requires a rule iOS doesn't have, it must be added to the
  **iOS** file first, then mirrored — not patched into Web directly.

## Review conditions

- Revisit if Cloud Functions ownership (Q19) lands somewhere that makes Web the more
  natural rules owner.
- Revisit if the platforms ever move to separate Firebase projects (they share
  `sector-9393c` today).
