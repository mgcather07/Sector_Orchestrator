# Migrations

Cross-platform migration plans — most importantly, **shared Realtime Database changes**.
Because all clients share one RTDB, a schema/path/rule change is never a single-repo edit;
it is a coordinated rollout across iOS, Android, Web, the rules, and Cloud Functions.

## When to write a migration

- Any change to a shared RTDB path, field name, value shape, or date convention.
- Any security-rule change or `.indexOn` change.
- Any Cloud Function change that alters written data shape or triggering behavior.
- Any cross-platform behavior change that must roll out in a specific order to keep older
  clients working.

## How to use

1. Copy [`TEMPLATE.md`](TEMPLATE.md) to `migrations/<id>-<slug>.md`.
2. Fill in affected contracts, repositories, RTDB paths, compatibility window, rollout
   order, backward compatibility, rollback, per-platform status, verification, and
   deployment approval.
3. Link it from the affected feature contract(s) and the RTDB contract.

## Rules of the road

- **Backward compatibility first.** Old app versions in the field keep reading/writing the
  old shape during the compatibility window. Additive changes before destructive ones.
- **Rules and functions are gated.** Until RTDB rules ownership is unified (open question
  Q19 / conflict C4), any rules deploy is a coordinated, approval-gated step in the plan.
- **No unapproved deployment.** A migration documents intent; shipping it requires explicit
  deployment approval from Michael.

## Index

_None yet. The first migration will likely accompany the resolution of the iOS↔Web rules
drift (C4) or a promoted feature that changes a shared path._
