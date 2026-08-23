# Task Registry

Cross-repository tasks and their status. **No implementation tasks are authorized yet.**
The rows below are **candidate** next actions surfaced by the bootstrap — each requires
Michael to approve scope (and, where noted, a status change) before it becomes an
`approved` task.

| Task | Parent feature | Authorized repos | Platform | Type | Status | Notes |
|---|---|---|---|---|---|---|
| _(candidate)_ Promote tournament-browsing to a canonical contract | tournament-browsing | Orchestrator only | multi (doc) | contract authoring | proposed | Recommended pilot. Inspect current iOS, reconcile vs Android/Web, fill platform matrix, Michael approves. |
| _(candidate)_ Verify Android tournament-browse filter wiring | tournament-browsing | Android_Sector (read-only) | android | verification | proposed | Confirm whether state/sort/50-mi filter is wired or still dead code (audit said dead; Phase-log said fixed). |
| _(candidate)_ Verify/resolve Android background geofencing | geofence-notifications | Android_Sector | android | verification | proposed | Conflict C1 — needs Michael's decision on background-location before any code. |
| _(candidate)_ Reconcile iOS↔Web RTDB rules drift | RTDB contract | TBD (Michael to assign owner) | multi | migration | proposed | Conflict C4 / Q19 — decide rules owner first; then a migration plan. |
| _(candidate)_ Update Android no-name fallback to "Member" | (terminology rule) | Android_Sector | android | bugfix | proposed | Conflict C3 — small mechanical fix once "Member" is confirmed final. |
| _(candidate)_ Confirm Android trips remain local-only | trips / cloud-sync | Android_Sector (read-only) | android | verification | proposed | Conflict C2 — confirm no trip cloud I/O in current `sync/UserDataSyncService.kt`. |

## Lifecycle

`proposed` → (Michael approves scope) → `approved` → `in_progress` → `in_review` →
`done` (or `blocked`). A task graduating to `approved` should get its own file if it is
more than a couple of lines, linked from this table.
