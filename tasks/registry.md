# Task Registry

Cross-repository tasks and their status. Most rows are `proposed` and require Michael to
approve scope before implementation; **0001 is done** — implemented (PR #13) and verified by 4 green instrumented tests (PR #14/#15). The `0001–0005` tasks were filed from the
**2026-08-23 Android parity audit** ([`../parity/android-parity.md`](../parity/android-parity.md));
each has its own file. Tasks **0008–0011** are proposed from the
[iOS delta since 2026-08-23](../parity/ios-delta-since-2026-08-23.md) (iOS PRs
#191–#204) — all client-only with **zero RTDB impact**. **Michael approved 0008–0011 on
2026-09-07**; each now has its own file. Implementation is authorized in
`Android_Sector` only (prepare a reviewable branch; no merge/deploy without
further authority).

| Task | Parent feature | Authorized repos | Platform | Type | Status | Notes |
|---|---|---|---|---|---|---|
| [0001](0001-android-boat-tracks-owner-scoping.md) Android boat-tracks owner scoping | boat-tracks | Android_Sector | android | bugfix (**CRITICAL** privacy) | **done** — PR #13/#14/#15 | No `ownerUid` / per-user filtering → a 2nd account on a shared device sees the 1st user's GPS history; recording never stops on sign-out. Also free cap 3→2. |
| [0002](0002-android-background-geofencing.md) Android background geofencing | geofence-notifications | Android_Sector | android | feature/bugfix | proposed | Conflict C1 — dormant (no `ACCESS_BACKGROUND_LOCATION`) → alerts foreground-only. **Needs Michael's Play-policy decision** before code. Supersedes the old verify candidate. |
| [0003](0003-android-apple-signin.md) Android Sign in with Apple | auth | Android_Sector | android | feature | **in_review** — PR #18 (⚠ needs Firebase Apple config) | `auth.md` requires it; absent on Android (corrects the stale "excluded" claim). Google Sign-In on deprecated GMS API — flagged for a follow-up. |
| [0004](0004-android-tournament-browse-filter-reachable.md) Android browse filter reachable | tournament-browse | Android_Sector | android | bugfix | **in_review** — PR #17 | Filter sheet is fully built but has **no entry point** → State filter + sort are dead functionality. Supersedes the old "verify filter wiring" candidate. |
| [0005](0005-android-analytics-instrumentation-parity.md) Android analytics parity | analytics (cross-cutting) | Android_Sector | android | feature | proposed | Redzone/tournament/guide/store events iOS logs are logged nowhere on Android — breaks cross-platform funnels. |
| [0006](0006-android-roster-order-and-host-add.md) Android roster order + host add-team | tournament-registration | Android_Sector | android | feature | **in_review** — PR #16 | Parity with **iOS PR #191**: number roster by registration order + sort filter; host adds a team for people without a smart device. **Unique id per host add** — the deterministic `{tid}_{uid}` id would overwrite. |
| [0007](0007-web-roster-order-and-host-add.md) Web roster order + host add-team | tournament-registration | Web_Sector | web | feature | **in_review** — main a3e08d6 | Parity with **iOS PR #191** (owner surface). Same two features; same unique-id caveat. |
| [0008](0008-android-score-consistency.md) Android cross-surface score consistency | conditions-engine | Android_Sector | android | bugfix | **in_review** — PR [#20](https://github.com/mgcather07/Android_Sector/pull/20) | Parity with **iOS #199/#200/#194**. Shared in-memory `ConditionsMemo` (10-min TTL, `%.3f` key) in `EngineApiClient.conditions()` that every tonight-score surface funnels through → identical bytes within the window, the "Cory bug". Compiles; `implemented_unverified` pending a two-device check. **Client-only, zero RTDB.** See [delta](../parity/ios-delta-since-2026-08-23.md). |
| [0009](0009-android-my-lakes-launch-preload.md) Android My Lakes preload at launch | my-lakes | Android_Sector | android | feature/perf | **in_review** — PR [#21](https://github.com/mgcather07/Android_Sector/pull/21) (stacked on #20) | Parity with **iOS #196**. `warmAll` at launch from MainActivity; `LakeScorer.request` two-phase (score first, snapshot enriches). Compiles; `implemented_unverified`. Client-only, zero RTDB. |
| [0010](0010-android-favorite-conditions.md) Android favorite conditions | conditions (dashboard) | Android_Sector | android | feature | **in_review** — PR [#22](https://github.com/mgcather07/Android_Sector/pull/22) | Parity with **iOS #195** (phone). `FavoriteConditionsStore` + star on each Conditions row → pinned strip on the home Tonight tab (reuses `rowData`). Compiles; `implemented_unverified`. Finding: Android's old tile-picker (`ConditionsTilesStore`) is parked/unused — iOS moved off it. iPad grid (#198) excluded. Client-only, zero RTDB. |
| [0012](0012-android-app-version-stamp.md) Android app-version stamp (Members screen) | admin-tools | Android_Sector | android | feature/bugfix | **in_review** — PR [#24](https://github.com/mgcather07/Android_Sector/pull/24) | Android wrote only `platform`, so Android accounts showed a blank version on the iOS master Members/Subscribers screen. Now stamps `users/{uid}/appVersion` + `appBuild` (strings) on sign-in, matching the iOS-owned contract. Compiles; `implemented_unverified`. Client-only, no schema change. |
| [0011](0011-android-conditions-detail-polish.md) Android conditions detail polish | weather-metric-detail / dashboard | Android_Sector | android | feature/bugfix | **in_review** — PR [#23](https://github.com/mgcather07/Android_Sector/pull/23) (stacked on #22) | Parity with **iOS #193/#194/#195**. **Mostly already at parity** — wind chart flush + now-boundary + scrub-both already present; paywall CTA already fixed (PR #19); fog awareness already an insight card (no night banner to adapt); water-temp has no range tabs (N-A). **One fix:** score-curve "Good" reference 60 → 65 (canonical band scheme). Compiles; `implemented_unverified`. Client-only, zero RTDB. |
| _(candidate)_ Promote tournament-browsing to a canonical contract | tournament-browsing | Orchestrator only | multi (doc) | contract authoring | proposed | Recommended pilot. Inspect current iOS, reconcile vs Android/Web, fill platform matrix, Michael approves. |
| _(candidate)_ Update Android no-name fallback to "Member" | (terminology rule) | Android_Sector | android | bugfix | proposed | Conflict C3 — small mechanical fix once "Member" is confirmed final. |
| _(candidate)_ Confirm Android trips remain local-only | trips / cloud-sync | Android_Sector (read-only) | android | verification | proposed | Conflict C2 — 2026-08-23 audit found **no** trip cloud I/O in current Android source; on-device confirm only. |
| _(resolved)_ Reconcile iOS↔Web RTDB rules drift | RTDB contract | iOS_Sector | multi | migration | **rules done** | Conflict C4 / Q19 — **rules half resolved**: iOS owns the canonical superset (ADR 0002); Web mirrors it. Remaining: Cloud Functions ownership. |

## Lifecycle

`proposed` → (Michael approves scope) → `approved` → `in_progress` → `in_review` →
`done` (or `blocked`). A task graduating to `approved` should get its own file if it is
more than a couple of lines, linked from this table.
