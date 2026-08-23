# Contracts

**Contracts are approved cross-platform behavior.** A contract describes what the
Sector product does across platforms in terms of user outcomes, product rules, and
shared data — not platform implementation details. Contracts become canonical only
after verification against current iOS behavior and approval by Michael; until then
they are drafts capturing the current best understanding.

Contracts are derived from, in evidence order:

1. Current iOS source & verified iOS behavior (the main reference).
2. Any approved orchestrator feature contract.
3. Current Android / Web source for the platform being described.
4. Current tests and build configuration.
5. Recently maintained Features & Specs.
6. Older parity plans and status docs.

## Files

| Contract | Scope |
|---|---|
| [`domain-model.md`](domain-model.md) | Shared domain concepts (Tournament, Redzone, Guide, …) and how each platform represents them. |
| [`realtime-database.md`](realtime-database.md) | The shared Firebase RTDB contract — 58 nodes, ownership, change control. **Highest-risk contract.** |
| [`authentication.md`](authentication.md) | Firebase Auth providers, guest mode, account lifecycle. |
| [`subscriptions.md`](subscriptions.md) | Premium entitlement model across StoreKit 2 / Play Billing / Web-unlock. |
| [`notifications.md`](notifications.md) | FCM push, local reminders, geofence alerts, Cloud-Function triggers. |

## Change control

Every contract carries a **change-control** expectation. The RTDB contract has the
strictest: any change is a gated cross-platform change (contract → impact → migration
→ tasks → compatibility → rules review → verification → deployment approval). See
[`../AGENTS.md`](../AGENTS.md) §9 and [`../migrations/`](../migrations/).

Each contract distinguishes: **current iOS behavior** · **confirmed shared behavior**
· **assumptions** · **platform differences** · **open questions** · **change-control
expectations**. Do not claim a Firebase service is used unless verified read-only.
