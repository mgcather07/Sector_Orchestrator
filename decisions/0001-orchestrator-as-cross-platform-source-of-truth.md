# 0001 — Orchestrator as cross-platform source of truth

- **Status:** accepted
- **Date:** 2026-08-22
- **Deciders:** Michael (with Claude Code establishing the foundation)

## Context

Sector ships as a native iOS app (the mature, primary product), an Android app, and a
Web app, all sharing one Firebase Realtime Database. Cross-platform work has been
coordinated through mirrored Features/Specs and periodic parity audits inside the iOS and
Android repos. That works for a single-feature push but has structural problems:

- No single place records **approved cross-platform intent** — what should be shared,
  what each platform builds, and what is intentionally excluded.
- Parity has been treated as a default goal ("catch Android up to iOS"), which risks
  chasing every iOS difference regardless of whether it belongs on other platforms.
- The shared RTDB contract lives implicitly in code and in two divergent `database.rules.json`
  copies (iOS and Web), with no owner — a real production risk (see conflict C4).
- "A spec says done" has been conflated with "it is verified done."

## Decision

Establish this orchestrator repository as the **canonical source of truth for approved
cross-platform intent**, with an explicit source-of-truth hierarchy:

1. **iOS** is the main source of truth for **current product behavior** and current RTDB
   usage.
2. The **orchestrator** is canonical for **approved cross-platform intent** — shared
   behavior, per-platform scope, adaptations, exclusions, migrations, and parity
   requirements. An approved orchestrator contract may intentionally differ from iOS for
   future work, and that difference is recorded.
3. **Android** and **Web** repos are the source of truth for their **own current
   implementations** only.
4. Existing **Features/Specs** are evidence of prior intent, never stronger than current
   iOS behavior or an approved contract.
5. The **RTDB contract** becomes canonical here only after verification and Michael's
   approval.

The orchestrator owns contracts, scope decisions, parity records, decisions, and
migrations. It does **not** own platform source code, credentials, or deployment.

**Parity is intentional, not automatic.** Every feature contract assigns an explicit
scope status per platform; no implementation work is created for an `excluded`,
`deferred`, or `not_evaluated` platform without Michael approving a status change. Scope
status (product intent) is kept strictly separate from assessment status (current
evidence).

Human review is a normal part of the workflow: behavior is inspected on iOS, documented
as a contract, scoped per platform, assessed for current evidence, and only then does
Michael authorize a bounded implementation task against specific repositories.

Existing parity documents are **migrated deliberately** into contracts over time — not
discarded — and remain as historical evidence.

## Alternatives considered

- **Keep coordinating in the iOS/Android repos via mirrored specs.** Rejected: no home
  for cross-platform intent, no separation of scope vs. assessment, and the RTDB
  ownership problem stays invisible.
- **Make the orchestrator the source of truth for behavior too (top-down specs).**
  Rejected: iOS is the mature, shipping product; declaring specs canonical over working
  iOS code would invert reality and invite drift.
- **A tooling-heavy orchestrator (database, website, CI bots, autonomous agents).**
  Rejected for now: adds operational surface with no proven need. Markdown + git + human
  review is sufficient and reversible.

## Consequences

- A new session reads the orchestrator before touching a platform, and works only in
  authorized repositories.
- Cross-platform changes (especially RTDB) follow an explicit change-control path.
- Some iOS features will be deliberately `excluded`/`adapted` on Android/Web with recorded
  rationale — parity is no longer assumed.
- Overhead: contracts and scope decisions must be authored and kept current; stale
  orchestrator docs would be as harmful as stale specs.

## Risks

- **Orchestrator drift** — if contracts lag reality, they mislead. Mitigation: contracts
  cite current source and are marked draft until verified; iOS remains the behavioral
  reference.
- **Dual RTDB rules ownership** (C4) persists until Q19 is decided; a bad deploy can
  overwrite rules. Mitigation: treat all rules deploys as coordinated, approval-gated.
- **Over-documentation** — mitigated by keeping the repo lightweight and only promoting
  features when there is a concrete need.

## Review conditions

Revisit this decision if: a second person joins development (roles/ownership change), the
backend changes (e.g. an approved Firestore evaluation), RTDB rules ownership is unified,
or the orchestrator's overhead is not paying for itself. Any move to introduce Firestore,
a hosted orchestrator service, or autonomous scheduled development requires its own ADR.
