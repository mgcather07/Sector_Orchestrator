# AGENTS.md — Working contract for Claude Code in the Sector Orchestrator

This file is the durable operating contract for any future Claude Code (or other
agent) session that works with Sector. Read it fully before changing anything in a
platform repository. It supplements — it does not override — the platform repos'
own `AGENTS.md`/`CLAUDE.md` files.

## Prime directives

1. **Read the orchestrator before you change a platform.** Start with
   [`README.md`](README.md), [`stack/inventory.md`](stack/inventory.md), the relevant
   [`contracts/`](contracts/), and any [`features/`](features/) contract for your area.
2. **iOS is the main behavioral reference.** To learn how a current Sector feature
   works, inspect current iOS source and its RTDB usage first. Older parity docs are
   evidence, not authority.
3. **Only touch explicitly authorized repositories.** A task names the repositories
   you may modify. Everything else is read-only. If scope is ambiguous, stop and ask.
4. **Inspect before editing.** Never edit a file you have not read. Never assume a
   file's contents from its name.
5. **Never assume parity.** A feature existing on iOS does not mean it should exist,
   or already exists, on Android or Web. Check the feature contract's platform matrix.

## Platform scope & assessment

6. **Respect each platform's scope status.** Do not implement, schedule, or "fix
   toward parity" a feature whose scope status for that platform is `excluded`,
   `deferred`, or `not_evaluated` without Michael approving a status change.
7. **Never modify an `excluded` platform** for that feature.
8. **Keep scope status separate from assessment status.** Scope = product intent;
   assessment = current evidence. Do not upgrade an assessment status to
   `verified_implemented` on the strength of a spec or a prior audit — that requires
   suitable verification (build/test/runtime), which a read-only pass cannot provide
   (use `implemented_unverified` or `verification_pending` instead).

## Backend & data

9. **Treat every Realtime Database change as a gated cross-platform change.** RTDB is
   shared by all clients. A schema/path/rule change requires: contract update →
   impact assessment → migration plan (if needed) → platform tasks →
   backward-compatibility review → security-rule review → compatibility verification
   → explicit deployment approval. See [`contracts/realtime-database.md`](contracts/realtime-database.md)
   and [`migrations/TEMPLATE.md`](migrations/TEMPLATE.md).
10. **Never introduce Firestore** (or any new datastore) without an approved
    architectural decision in [`decisions/`](decisions/).
11. **Never write Firebase data, rules, or config** as part of orchestrator work, and
    never change Firebase infrastructure without explicit deployment approval.

## Secrets, safety & scope

12. **Never include secrets.** No API keys, service-account JSON, `.env` values,
    keystores, signing certs, provisioning profiles, or tokens — in this repo or in
    any change. Reference sensitive config **by filename only**.
13. **Never deploy, merge, release, commit, or push without appropriate authority.**
    Do not commit or push unless Michael explicitly requests it.
14. **Avoid changing iOS during Android or Web work** unless the task authorizes it.
    iOS is the reference; drifting it mid-task corrupts the source of truth.
15. **Preserve unrelated user changes.** Do not reformat, move, rename, or "clean up"
    files outside your task. Do not delete or rewrite existing Features/Specs folders.
16. **Verify in proportion to risk.** A doc edit needs a link check; a data-touching
    or schema change needs real build/test/compatibility verification.
17. **Stop before** destructive actions, schema migrations, production changes,
    repository creation, credential handling, or any materially expanded scope — and
    ask. Record choices you cannot make as open questions rather than inventing them.

## When work is done — report

Report **all** of the following, honestly, including what you did _not_ do:

- Which repositories were **authorized**, which were **inspected**, which were **changed**.
- What **iOS behavior** you referenced and which **contracts** you consulted.
- The **verification** you performed and its **results** (including failures).
- Any **Realtime Database impact** and any **deployment impact**.
- **Unresolved issues** and work you **deliberately did not perform**.
- **Commit/push/PR status** (default: none, unless Michael asked).

## Completion report

Use this exact format at the end of a session:

```markdown
## Completion report
- Authorized repositories:
- Repositories inspected:
- Repositories changed:
- iOS behavior referenced:
- Contracts consulted:
- Changes made:
- Verification performed:
- Verification results:
- Realtime Database impact:
- Deployment impact:
- Unresolved issues:
- Work deliberately not performed:
- Commit/push/PR status:
```
