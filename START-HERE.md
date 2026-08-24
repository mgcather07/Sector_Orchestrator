# START HERE — cross-repo session kickoff

Open this **first** whenever a session will touch more than one Sector repo. For focused
work inside a single app, just work in that repo — you don't need this.

This is the operational companion to [`AGENTS.md`](AGENTS.md) (the *why* and the rules).
This file is the *how*: read order, the repos, and each repo's git flow.

---

## 1. Read in this order (~2 minutes)

1. **This file.**
2. [`AGENTS.md`](AGENTS.md) — the working contract + safety rules. Non-negotiable.
3. [`stack/inventory.md`](stack/inventory.md) — every repo, local path, tech, build/test entry points.
4. The **contract(s)/parity for your area** — [`contracts/`](contracts/), and
   [`parity/android-parity.md`](parity/android-parity.md) for Android gaps.
5. [`tasks/registry.md`](tasks/registry.md) — is your work an approved task, or still `proposed`?
6. [`docs/open-questions.md`](docs/open-questions.md) — is there an unresolved decision in your path?

## 2. The repos (paths on this Mac — see `stack/inventory.md` if they differ)

| Repo | Local path | Role | Git flow |
|---|---|---|---|
| **iOS** (`iOS_Sector`) | `~/Desktop/Development/iOS/Sector` | **Behavioral source of truth**; owns RTDB rules + Cloud Functions | git-guard · trunk `Michael-Master` · **branch → PR → merge** |
| **Android** (`Android_Sector`) | `~/Desktop/Development/Android/SectorAndroid` | Android client; mirrors iOS | git-guard · trunk `Michael-Master` · **branch → PR → merge** |
| **Web** (`Web_Sector`) | `~/Desktop/Development/Web/Sector-Web` | Web client; mirrors iOS RTDB rules | **no git-guard — commits to `main` directly** |
| **Orchestrator** (this) | `~/Desktop/Development/Orchestrator/Sector` | Coordination: contracts, parity, ADRs, tasks | commits to `main` directly |

## 3. Pick your "home" repo

Only **one** repo's `CLAUDE.md` / hooks / skills auto-load in a session (the one you launch
from). Everything else you read **manually**.

- **Default home: iOS** — it's the source of truth and where most code work happens.
- **Pure-coordination session** (writing contracts/decisions/tasks, no app code): launch from
  the orchestrator.
- **Before editing any non-home repo, read its `CLAUDE.md` first** (each points back here).

## 4. Golden rules (condensed — full set in `AGENTS.md`)

- **iOS is the behavioral source of truth.** Learn how a feature works from current iOS source.
- **iOS owns `database.rules.json`** ([ADR 0002](decisions/0002-ios-owns-rtdb-rules.md)). Edit
  rules in iOS → mirror the file to Web → deploy **once, from iOS**. Never hand-edit Web's copy.
- **Never introduce Firestore** or any new datastore (RTDB-only, by decision).
- **Parity is intentional.** A feature on iOS is *not* automatically due on Android/Web —
  check [`parity/`](parity/) and the feature's scope status before building it elsewhere.
- **Tasks name their authorized repos; touch nothing else.** A `proposed` task is not
  authorization — it needs Michael's approval to become `approved`.
- **Commit/push only when Michael asks**, then follow *that repo's* git flow (below).
- **Never commit secrets** (`.env`, service-account keys, keystores, tokens).

## 5. Per-repo git flow (they differ — this matters)

- **iOS_Sector / Android_Sector** — git-guard is ON. Trunk is `Michael-Master`. **Never commit
  to trunk directly.** Flow: `git checkout Michael-Master && git pull --ff-only` → `git checkout
  -b <branch>` → commit → `git push -u` → `gh pr create --base Michael-Master` → `gh pr merge
  <branch> --merge --admin` → pull trunk, delete branch. git-guard may strand edits on a
  `wip/…` autosave branch — recover with `git log -g -S "<unique string>"`.
- **Web_Sector** — no git-guard; commits go straight to `main`. It often carries a **large
  amount of uncommitted work** — always `git add` **only your specific files**, never `-A`
  blindly, and scan for secrets before committing.
- **Sector_Orchestrator** — docs only; commit to `main` directly.

## 6. Big or parallel cross-repo work

Don't drown the main session. Fan out with **subagents / a workflow** (as the 2026-08-23
Android parity audit did — 9 read-only auditors across ~83k LOC, returning just the summary),
then act on the result in the main thread. The orchestrator's compact contracts/parity mean
you read *those*, not the raw code, to stay coordinated.

---

_When work crosses repos, the order is always: read the orchestrator → identify the authorized
repos → inspect before editing → follow each repo's git flow → report what changed where._
