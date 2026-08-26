# Task Registry

Cross-repository tasks and their status. Most rows are `proposed` and require Michael to
approve scope before implementation; **0001 is done** — implemented (PR #13) and verified by 4 green instrumented tests (PR #14/#15). The `0001–0005` tasks were filed from the
**2026-08-23 Android parity audit** ([`../parity/android-parity.md`](../parity/android-parity.md));
each has its own file.

| Task | Parent feature | Authorized repos | Platform | Type | Status | Notes |
|---|---|---|---|---|---|---|
| [0001](0001-android-boat-tracks-owner-scoping.md) Android boat-tracks owner scoping | boat-tracks | Android_Sector | android | bugfix (**CRITICAL** privacy) | **done** — PR #13/#14/#15 | No `ownerUid` / per-user filtering → a 2nd account on a shared device sees the 1st user's GPS history; recording never stops on sign-out. Also free cap 3→2. |
| [0002](0002-android-background-geofencing.md) Android background geofencing | geofence-notifications | Android_Sector | android | feature/bugfix | proposed | Conflict C1 — dormant (no `ACCESS_BACKGROUND_LOCATION`) → alerts foreground-only. **Needs Michael's Play-policy decision** before code. Supersedes the old verify candidate. |
| [0003](0003-android-apple-signin.md) Android Sign in with Apple | auth | Android_Sector | android | feature | proposed | `auth.md` requires it; absent on Android (corrects the stale "excluded" claim). Google Sign-In on deprecated GMS API — flagged for a follow-up. |
| [0004](0004-android-tournament-browse-filter-reachable.md) Android browse filter reachable | tournament-browse | Android_Sector | android | bugfix | proposed | Filter sheet is fully built but has **no entry point** → State filter + sort are dead functionality. Supersedes the old "verify filter wiring" candidate. |
| [0005](0005-android-analytics-instrumentation-parity.md) Android analytics parity | analytics (cross-cutting) | Android_Sector | android | feature | proposed | Redzone/tournament/guide/store events iOS logs are logged nowhere on Android — breaks cross-platform funnels. |
| [0006](0006-android-roster-order-and-host-add.md) Android roster order + host add-team | tournament-registration | Android_Sector | android | feature | proposed | Parity with **iOS PR #191**: number roster by registration order + sort filter; host adds a team for people without a smart device. **Unique id per host add** — the deterministic `{tid}_{uid}` id would overwrite. |
| [0007](0007-web-roster-order-and-host-add.md) Web roster order + host add-team | tournament-registration | Web_Sector | web | feature | proposed | Parity with **iOS PR #191** (owner surface). Same two features; same unique-id caveat. |
| _(candidate)_ Promote tournament-browsing to a canonical contract | tournament-browsing | Orchestrator only | multi (doc) | contract authoring | proposed | Recommended pilot. Inspect current iOS, reconcile vs Android/Web, fill platform matrix, Michael approves. |
| _(candidate)_ Update Android no-name fallback to "Member" | (terminology rule) | Android_Sector | android | bugfix | proposed | Conflict C3 — small mechanical fix once "Member" is confirmed final. |
| _(candidate)_ Confirm Android trips remain local-only | trips / cloud-sync | Android_Sector (read-only) | android | verification | proposed | Conflict C2 — 2026-08-23 audit found **no** trip cloud I/O in current Android source; on-device confirm only. |
| _(resolved)_ Reconcile iOS↔Web RTDB rules drift | RTDB contract | iOS_Sector | multi | migration | **rules done** | Conflict C4 / Q19 — **rules half resolved**: iOS owns the canonical superset (ADR 0002); Web mirrors it. Remaining: Cloud Functions ownership. |

## Lifecycle

`proposed` → (Michael approves scope) → `approved` → `in_progress` → `in_review` →
`done` (or `blocked`). A task graduating to `approved` should get its own file if it is
more than a couple of lines, linked from this table.
