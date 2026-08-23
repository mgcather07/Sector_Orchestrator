# Sector Orchestrator

The **private cross-platform coordination layer** for Sector — an application for
fishing, bowfishing, and outdoor-sports communities. This repository is where the
_intent_ of the Sector product lives: what behavior is shared across platforms,
which platform builds what, what the shared Firebase Realtime Database contract is,
and how cross-platform changes are reviewed and rolled out.

> Sector is a **bowfishing** app. Users are **bowfishers**, never "anglers." The
> current no-name display fallback is **"Member"** (changed from "Bowfisher" on
> 2026-07-19 on iOS). This is a product rule, not a style preference — see
> [`contracts/domain-model.md`](contracts/domain-model.md).

## What this is

- A coordination repository holding **feature contracts**, **platform scope
  decisions**, **shared domain and database contracts**, **architectural
  decisions**, **cross-platform migrations**, **bounded tasks**, and **Android/Web
  parity assessments**.
- The **canonical source of truth for approved cross-platform intent** — the
  reviewed answer to "what should the product do across platforms, and who builds it."
- The place a cold Claude Code session reads **before** touching any platform repo.

## What this is **not**

- **Not** a customer-facing application.
- **Not** the home of production iOS, Android, or Web source code — those live in
  their own repositories and remain the source of truth for _their own_ implementations.
- **Not** a store of credentials, secrets, service-account keys, signing
  certificates, keystores, or deployment tokens.
- **Not** a build system, hosted website, custom database, deployment pipeline, or
  autonomous agent runner. It is Markdown + small YAML metadata under git.

## Source-of-truth hierarchy

| Source | Authority |
|---|---|
| **iOS repository** | Main source of truth for **currently implemented product behavior** and current Realtime Database usage. Primarily developed by Michael. |
| **Orchestrator (this repo)** | Canonical source of truth for **approved cross-platform intent** — shared behavior, platform scope, adaptations, exclusions, migrations, and parity requirements. |
| **Android repository** | Source of truth for the **current Android implementation** only — not for shared product intent. |
| **Web repository** | Source of truth for the **current Web implementation** only — not for shared product intent. |
| **Existing Features & Specs** (in the iOS/Android repos) | Evidence of prior intent and parity planning. **Not** stronger than current iOS behavior or an approved orchestrator contract. |
| **Realtime Database contract** (here) | Becomes canonical **after** verification against real implementation and approval by Michael. |

If an existing parity document conflicts with current iOS behavior, **iOS wins** as
evidence of how Sector presently works — but the conflict is _recorded_
([`parity/unresolved-conflicts.md`](parity/unresolved-conflicts.md)), never silently
rewritten. If an **approved orchestrator contract** intentionally differs from iOS,
the contract controls future cross-platform work, and whether iOS should eventually
change is recorded explicitly.

iOS being the main source of truth does **not** mean every iOS feature belongs on
Android or Web. The orchestrator decides whether and how iOS behavior applies elsewhere.

## The platforms and the backend

| Component | Repository |
|---|---|
| iOS (native, Swift/SwiftUI) | https://github.com/mgcather07/iOS_Sector.git |
| Android (Kotlin/Jetpack Compose) | https://github.com/mgcather07/Android_Sector.git |
| Web (Next.js/TypeScript) | https://github.com/mgcather07/Web_Sector.git |
| Orchestrator (this repo) | https://github.com/mgcather07/Sector_Orchestrator.git |

**Shared backend: Firebase Realtime Database (RTDB).** All platforms read and write
the same RTDB paths so the same server-side Cloud Functions fire regardless of
client. Firestore is **not** part of this architecture and must not be introduced
without an approved architectural decision. See
[`contracts/realtime-database.md`](contracts/realtime-database.md).

## Scope status vs. assessment status

These are **two different axes** and must never be conflated:

- **Scope status** = what the product _intends_ to support on a platform:
  `not_evaluated` · `required` · `adapted` · `deferred` · `excluded` · `implemented`.
  (`adapted`/`deferred`/`excluded` require a rationale.)
- **Assessment status** = what the current repository _evidence_ shows:
  `verified_implemented` · `implemented_unverified` · `partial` · `documented_only`
  · `missing` · `intentionally_different` · `obsolete` · `unknown` · `verification_pending`.

A spec claiming "complete" is **never** enough to mark something
`verified_implemented`. See [`features/TEMPLATE.md`](features/TEMPLATE.md) for the
platform matrix that carries both.

## Cross-platform operating principle

**Parity is intentional, not automatic.** A feature added to iOS is **not**
automatically scheduled or built on Android or Web. Every feature contract assigns
an explicit scope status per platform, and no implementation work is created for an
`excluded`, `deferred`, or `not_evaluated` platform without Michael approving a
status change.

## Repository map

```text
Sector_Orchestrator/
├── README.md                     ← you are here
├── AGENTS.md                     ← durable working contract for Claude Code sessions
├── stack/
│   └── inventory.md              ← components, repos, tech, status, build/test entry points
├── contracts/                    ← approved cross-platform behavior contracts
│   ├── README.md
│   ├── domain-model.md           ← shared domain concepts (Tournament, Redzone, …)
│   ├── realtime-database.md      ← the shared RTDB contract (58 top-level nodes)
│   ├── authentication.md
│   ├── subscriptions.md
│   └── notifications.md
├── features/                     ← feature contracts (approved cross-platform intent)
│   ├── README.md
│   ├── TEMPLATE.md
│   └── registry.md               ← candidate feature list (mostly not_evaluated)
├── parity/                       ← Android/Web parity reconciliation vs iOS
│   ├── README.md
│   ├── android-parity.md
│   ├── evidence-register.md
│   └── unresolved-conflicts.md
├── platforms/
│   ├── ios.md
│   ├── android.md
│   └── web.md
├── decisions/                    ← architectural decision records (ADRs)
│   ├── README.md
│   └── 0001-orchestrator-as-cross-platform-source-of-truth.md
├── migrations/                   ← cross-platform / RTDB migration plans
│   ├── README.md
│   └── TEMPLATE.md
├── tasks/                        ← bounded, authorized implementation tasks
│   ├── README.md
│   └── registry.md
└── docs/
    ├── workflow.md               ← the 11 core workflows
    └── open-questions.md         ← unresolved decisions awaiting Michael
```

## How a cold Claude Code session should begin

1. Read [`AGENTS.md`](AGENTS.md) — the working contract and safety rules.
2. Read [`stack/inventory.md`](stack/inventory.md) to locate the repositories and
   their local paths, tech, and build/test entry points.
3. Read the relevant [`contracts/`](contracts/) and any
   [`features/`](features/) contract for the area you're working in.
4. Use **iOS** as the behavioral reference for how a current feature works; verify
   against current iOS source before treating any older doc as authoritative.
5. Identify the **explicitly authorized** repositories for your task, inspect before
   editing, and respect each platform's scope status.
6. Never introduce Firestore, secrets, deployments, schema migrations, or new
   repositories without an approved decision.

## Unresolved decisions

The foundation intentionally records open questions rather than inventing answers.
See [`docs/open-questions.md`](docs/open-questions.md). Highlights awaiting Michael:

- Confirm all platforms share **one** Firebase project/database instance, and how
  dev/staging/prod are separated (iOS + Android show a runtime dev/prod switch).
- Confirm iOS subscription provider (StoreKit 2 vs RevenueCat — both appear present
  in the iOS repo) vs Android (Google Play Billing) vs Web (undetermined).
- Confirm which Firebase products are in use beyond RTDB/Auth/FCM/Storage/Functions.
- Confirm the intended parity level for the first Android and first Web releases.
- Reconcile the iOS↔Web RTDB rules drift (`offerCodes` vs `playAccountTokens`,
  entitlement-write logic) — see [`parity/unresolved-conflicts.md`](parity/unresolved-conflicts.md).

---

_This repository is a coordination layer. It records decisions and intent; it does
not itself ship product code. Keep it lightweight, human-readable, and honest about
what has and has not been verified._
