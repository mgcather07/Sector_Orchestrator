# Workflows

The task-driven workflows this orchestrator supports. Each starts from iOS behavior,
passes through an orchestrator contract with explicit per-platform scope, separates scope
from assessment, and ends with Michael authorizing bounded work in specific repositories.

Throughout: **inspect before editing**, **modify only authorized repositories**, **never
assume parity**, **treat RTDB changes as gated**, and **never deploy/commit/push without
authority**.

## 1. Extract current iOS behavior into an orchestrator contract

1. Inspect the current iOS implementation of the feature (source is the reference).
2. Note its RTDB paths/fields and any product rules (gating, ordering, hard rules).
3. Copy [`../features/TEMPLATE.md`](../features/TEMPLATE.md) → `features/<id>.md`; fill
   summary, user outcome, current iOS behavior, product rules, shared behavior.
4. Assign a per-platform scope status (start `not_evaluated` for platforms not yet decided).
5. Record assessment status from current evidence (read-only ⇒ `implemented_unverified` /
   `verification_pending`).
6. Michael reviews and approves the platform matrix.

## 2. Migrate an existing Android parity spec

1. Read the spec (`iOS_Sector/specs/<slug>.md`) and Android's audit rows for it.
2. Reconcile against **current** iOS and Android source (source beats the stale audit).
3. Author the contract; record any conflicts in
   [`../parity/unresolved-conflicts.md`](../parity/unresolved-conflicts.md).
4. Leave the original spec/audit in place as historical evidence (do not delete/rewrite).

## 3. Declare a feature iOS-only

1. Confirm the feature exists on iOS and there is a reason it should not be on Android/Web.
2. In the contract, set Android and Web `scope_status: excluded` with a **rationale**.
3. Michael approves. No implementation tasks are created for excluded platforms.

> Example: iPad/large-screen 2-column layouts — `excluded` on Android/Web (iOS-only).

## 4. Adapt an iOS feature for Android

1. Document the shared user outcome and the platform-appropriate adaptation.
2. Set Android `scope_status: adapted` with a rationale (e.g. Google Maps not MapKit).
3. Keep the **shared RTDB contract identical** — adapt presentation, not data.

> Example: redzone map — Android renders with Google Maps overlays (`adapted`), writing
> the same `redzones/{rzId}` shape.

## 5. Adapt an iOS feature for Web

1. Same as #4 for Web idioms (Next.js, Mapbox GL), noting Web capability limits.
2. Where a capability is infeasible on Web, use `excluded` with rationale.

> Example: background geofence — `excluded` on Web (background geofencing is not reliable
> on the Web); the redzone *map* is still `adapted`/`required`.

## 6. Schedule Android parity work

1. Ensure an approved contract with Android `scope_status: required` (or `adapted`).
2. Create a bounded task in [`../tasks/registry.md`](../tasks/registry.md): authorized repo
   = `Android_Sector`, iOS behavior reference, acceptance criteria, verification method.
3. Michael approves the task. Claude implements in Android only, prepares a reviewable
   branch, does not deploy/merge.

> Example: "Implement the approved tournament-calendar contract on Android. Use iOS as
> the behavioral reference. Do not modify iOS, Web, Firebase contracts, or deployment
> configuration."

## 7. Schedule Web parity work

Same as #6 with authorized repo = `Web_Sector`. Note Web has **no automated tests** —
verification leans on build + manual/route checks; record that honestly.

## 8. Change a Realtime Database contract safely

1. Update [`../contracts/realtime-database.md`](../contracts/realtime-database.md).
2. Impact-assess across iOS/Android/Web (who reads/writes the path).
3. Write a migration plan ([`../migrations/TEMPLATE.md`](../migrations/TEMPLATE.md)):
   compatibility window, rollout order, backward compatibility, rollback.
4. Create per-platform tasks. Review security rules (**both** rules copies until Q19/C4 is
   resolved).
5. Verify read/write compatibility, including an older client.
6. Get explicit deployment approval before rules/functions/data ship.

## 9. Complete implementation and update assessment status

1. After a task's branch is built/tested, update the feature contract's **assessment
   status** for that platform based on the verification actually performed.
2. Only `verified_implemented` if suitable verification (build/test/runtime) supports it;
   otherwise `implemented_unverified` / `verification_pending`.
3. Record the outcome in the task's completion record and the contract's decision history.

## 10. Record an intentional difference from iOS

1. When an approved contract deliberately differs from current iOS behavior, state the
   difference and set the platform's assessment to `intentionally_different`.
2. Record whether iOS should eventually change, stay platform-specific, or is exempted.

> Example: Android's simplified conditions engine — `intentionally_different` with a note
> that scores may diverge from the shared Sector_Engine numbers.

## 11. Reverse an earlier excluded / deferred / unevaluated decision

1. Michael approves the scope change (e.g. Web `excluded` → `required`).
2. Update the feature contract's platform matrix and its decision history with the date
   and reason.
3. Only then create implementation tasks. **Never** implement toward a reversed decision
   before the scope change is approved.
