# Parity

How the orchestrator reconciles current iOS behavior, the existing Features/Specs, and
the current Android/Web implementations.

## The model

- **iOS is the main source of truth** for how a current Sector feature behaves.
- **Orchestrator feature contracts** are the approved cross-platform intent — an
  approved contract may intentionally differ from iOS, and that difference is recorded.
- **Product scope** (what a platform should support) is separate from **implementation
  assessment** (what current evidence shows). Never conflate them.
- **Parity does not require identical interfaces.** Android keeps Google Maps; Web uses
  Mapbox GL; Web excludes background geofencing. Parity means the same *user outcomes*
  and *shared data contract*, adapted to each platform's idioms.

## Evidence hierarchy

When reconciling behavior and parity, weigh evidence in this order:

1. Current iOS source code and verified iOS behavior.
2. Approved orchestrator feature contracts.
3. Current Android/Web source and verified behavior for the platform being assessed.
4. Current tests and build configuration.
5. Recently maintained Features & Specs.
6. Older parity plans and status documents.
7. Assumptions inferred from filenames or incomplete code.

When evidence conflicts, **record the conflict** — never silently pick or rewrite one
side. See [`unresolved-conflicts.md`](unresolved-conflicts.md).

## How existing Features & Specs feed the orchestrator

Michael maintains a mirrored **spec set** (~40 features) in `iOS_Sector/specs/` (the
source) and `Android_Sector/specs/` (a generated mirror, synced by `ops/sync-specs.sh`).
Android additionally has two parity documents:

- `docs/PARITY-AUDIT.md` (2026-07-15, ~880 lines) — the detailed feature-by-feature
  audit with file:line evidence and a "shipped this pass" section. **The key artifact.**
- `docs/iOS-PARITY-ROADMAP.md` (2026-05-29) — an older phase roadmap, now stale.

These are **important migration inputs and historical evidence**. After a feature is
promoted into an orchestrator contract, the old docs remain as evidence — they are not
deleted or rewritten. Where they conflict with current iOS behavior or current source,
current evidence wins and the conflict is recorded.

> **Staleness caveat:** both Android parity docs predate the current source. Android has
> matured **past** its own 2026-07-15 audit (e.g. Chum Bucket and redzone submission are
> now in source), and iOS handoffs run to **2026-08-06**. Trust source over both docs.

## How Android parity is evaluated

See [`android-parity.md`](android-parity.md) for the per-feature table (scope status,
assessment status, gaps, verification needed, next action, confidence). Because this was
a **read-only** pass, no build/runtime verification was performed — claims that a spec or
audit says "done" are recorded as `implemented_unverified` or `verification_pending`, not
`verified_implemented`.

## Files

- [`android-parity.md`](android-parity.md) — Android vs iOS reconciliation table.
- [`evidence-register.md`](evidence-register.md) — where each conclusion's evidence lives.
- [`unresolved-conflicts.md`](unresolved-conflicts.md) — contradictions needing a decision.

_Web parity has not been formally assessed in this bootstrap beyond the platform
inventory in [`../platforms/web.md`](../platforms/web.md); a Web parity table is a future
addition once Web scope per feature is decided._
