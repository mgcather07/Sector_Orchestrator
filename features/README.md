# Features

A **feature contract** is the approved cross-platform intent for one feature: the
shared user outcome and product rules, plus an explicit **scope status** and
**assessment status** for each platform. Contracts describe outcomes and rules — not
SwiftUI implementation details translated verbatim.

## How to use

- To author a new contract, copy [`TEMPLATE.md`](TEMPLATE.md) to `features/<feature-id>.md`.
- Every contract carries a **platform matrix** with both scope and assessment status.
- A contract is `draft` until Michael approves its platform matrix; only then does it
  become canonical for future cross-platform work.

## Scope vs. assessment (never conflate)

- **Scope status** (product intent): `not_evaluated` · `required` · `adapted` ·
  `deferred` · `excluded` · `implemented`. `adapted`/`deferred`/`excluded` require a
  rationale.
- **Assessment status** (current evidence): `verified_implemented` ·
  `implemented_unverified` · `partial` · `documented_only` · `missing` ·
  `intentionally_different` · `obsolete` · `unknown` · `verification_pending`.

A spec claiming "done" never justifies `verified_implemented`; that needs suitable
verification, which a read-only pass cannot provide.

## Candidate features

The list of candidate features lives in [`registry.md`](registry.md). **Candidates stay
`not_evaluated` until Michael confirms scope** — extracting a spec from iOS or the
existing Features/Specs does not create a cross-platform commitment.

## Relationship to platform specs

iOS ships ~40 feature **specs** (in `iOS_Sector/specs/`, mirrored into `Android_Sector/
specs/`). Those specs are the raw material; a feature **contract** here is the reviewed,
scoped, cross-platform version. The registry maps candidates back to their source specs.
