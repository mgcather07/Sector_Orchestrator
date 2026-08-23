<!--
Copy this file to features/<feature-id>.md to author a feature contract.
A feature contract is APPROVED CROSS-PLATFORM INTENT — it describes the shared
user outcome and product rules, and assigns an explicit scope status per platform.
Do NOT translate SwiftUI implementation details verbatim; document outcomes & rules.
Delete these comments and any sections that genuinely do not apply.
-->

# Feature Contract: <Feature Name>

```yaml
feature_id: <kebab-case-id>          # stable identifier, e.g. tournament-browsing
name: <Human Readable Name>
status: draft                        # draft | approved | superseded
owner: Michael
approved_by:                         # who approved this contract (blank until approved)
approved_on:                         # YYYY-MM-DD
source_specs:                        # original Features/Specs docs this derives from
  - iOS: specs/<slug>.md
  - Android: specs/<slug>.md
platforms:
  ios:
    scope_status: not_evaluated      # not_evaluated|required|adapted|deferred|excluded|implemented
    assessment_status: unknown       # see legend below
    # rationale: required only for adapted|deferred|excluded
  android:
    scope_status: not_evaluated
    assessment_status: unknown
  web:
    scope_status: not_evaluated
    assessment_status: unknown
```

**Scope status** (product intent): `not_evaluated` · `required` · `adapted` ·
`deferred` · `excluded` · `implemented`. `adapted`/`deferred`/`excluded` **must**
carry a `rationale`.
**Assessment status** (current evidence): `verified_implemented` ·
`implemented_unverified` · `partial` · `documented_only` · `missing` ·
`intentionally_different` · `obsolete` · `unknown` · `verification_pending`.

## Summary
_One paragraph: what the feature is, in product terms._

## User outcome
_What the bowfisher (or guide/store owner/admin) is able to accomplish._

## Current iOS behavior
_How this presently works on iOS — the behavioral reference. Cite iOS source/spec._

## Product rules
_The rules that hold regardless of platform (gating, ordering, validation, hard
rules like "no 'angler' in user-facing copy", "pressure shown in inHg")._

## In scope
## Out of scope

## Shared behavior
_The behavior every in-scope platform must honor identically (usually the RTDB
data contract, field names, server-triggering writes)._

## Platform matrix

| Platform | Scope status | Assessment status | Notes |
|---|---|---|---|
| iOS | | | |
| Android | | | |
| Web | | | |

## Platform adaptations
_Where a platform intentionally differs (e.g. Android keeps Google Maps; Web has no
background geofencing). Each adaptation needs a rationale._

## Realtime Database impact
_RTDB paths/fields read or written; whether this changes the shared contract. Link
to [`../contracts/realtime-database.md`](../contracts/realtime-database.md)._

## Authentication impact
## Subscription impact
## Notification impact
## Analytics impact
## Accessibility expectations
## Offline behavior
## Error behavior

## Acceptance criteria
_Testable, platform-appropriate. Not "matches iOS pixel-for-pixel."_

## Verification plan
_How completion is verified per platform (build, unit test, on-device/runtime).
A read-only pass yields `implemented_unverified` / `verification_pending`._

## Dependencies
## Risks
## Open questions
## Decision history
_Dated entries: what changed, who decided._

## Original parity-document references
_Repo-relative paths to the source Features/Specs and handoff docs this is built from._
