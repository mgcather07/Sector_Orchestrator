<!-- Copy to migrations/<id>-<slug>.md. Delete guidance comments and N/A sections. -->

# Migration: <Title>

```yaml
migration_id: <NNNN-slug>
status: draft            # draft | approved | in_progress | complete | rolled_back
owner: Michael
opened: <YYYY-MM-DD>
```

## Summary
_One paragraph: what changes and why._

## iOS behavior being changed
_The current iOS behavior/data this migration alters (the reference). Cite source._

## Affected contracts
_Links to feature contracts and [`../contracts/realtime-database.md`](../contracts/realtime-database.md)._

## Affected repositories
- [ ] iOS_Sector
- [ ] Android_Sector
- [ ] Web_Sector
- [ ] RTDB rules (currently in iOS + Web — see conflict C4)
- [ ] Cloud Functions (iOS `functions/` and/or Web `functions/`)

## Affected Realtime Database paths
_Exact node paths, fields, value shapes, date conventions, and `.indexOn` changes._

## Compatibility window
_How long old and new shapes must coexist; which app versions are still in the field._

## Rollout order
_The ordered steps (e.g. add new field → dual-write → migrate readers → deploy rules →
backfill → remove old field). Order that keeps older clients working._

## Backward compatibility
_How already-installed clients keep working during the window. Additive-first plan._

## Data migration
_Any backfill/transform needed; who runs it; idempotency; how much data._

## Security-rule impact
_Rule changes required; review of BOTH rules copies until ownership is unified._

## Rollback
_How to revert safely at each step; the point of no return, if any._

## Platform status

| Platform | Change needed | Status | Notes |
|---|---|---|---|
| iOS | | not_started | |
| Android | | not_started | |
| Web | | not_started | |
| Rules | | not_started | |
| Functions | | not_started | |

## Verification
_Per-platform read/write round-trip checks; compatibility test with an old client;
build/test evidence. Proportional to risk._

## Deployment approval
_Explicit Michael approval required before rules/functions/data ship. Record date + scope._

## Completion record
_What shipped, when, verification results, and any follow-ups. Fill on completion._
