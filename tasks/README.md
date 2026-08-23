# Tasks

Bounded, authorized units of cross-repository work. A task is how approved intent becomes
action: it names exactly which repositories may be changed, references the iOS behavior
and contracts, and states how completion is verified and who reviews it.

This is a **lightweight Markdown format**, not a task-management application. The registry
is a table; individual tasks can be a row or, when larger, their own file
(`tasks/<id>-<slug>.md`) linked from the registry.

## Task format

```yaml
task_id: <NNNN-slug>
parent_feature: <feature-id or contract link>
authorized_repositories:            # the ONLY repos this task may modify
  - Android_Sector
platform: android                   # ios | android | web | multi
ios_behavior_reference: <spec/source path>   # the behavioral reference
status: proposed                    # proposed | approved | in_progress | in_review | done | blocked
deployment_authority: none          # none unless Michael grants it
review_requirement: Michael approves the branch/PR
```

Plus prose sections: **Scope**, **Out of scope**, **Contract references**,
**Dependencies**, **Acceptance criteria**, **Verification method**, **Completion record**.

## Rules

- A task modifies **only** its `authorized_repositories`. Everything else is read-only.
- No task is created for an `excluded`/`deferred`/`not_evaluated` platform without Michael
  approving a scope change first.
- No commit/push/PR/merge/deploy without the stated authority. Default is: prepare a
  reviewable branch, Michael reviews.
- RTDB-touching tasks must reference a migration plan (see [`../migrations/`](../migrations/)).

## Index

See [`registry.md`](registry.md). No implementation tasks are authorized yet — the
foundation only records **candidate** next actions.
