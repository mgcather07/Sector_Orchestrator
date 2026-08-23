# Decisions (ADRs)

Architectural Decision Records for cross-platform Sector choices. An ADR captures a
decision that is expensive to reverse or that multiple platforms depend on — so the
reasoning survives past the moment it was made.

## When to write one

- Choosing/adding a shared technology or backend (e.g. "introduce Firestore" — which
  would require an ADR before any code).
- Changing the source-of-truth or ownership model.
- Deciding a cross-platform data contract direction or a platform exclusion that affects
  the whole product.

## Format

Number sequentially (`NNNN-title.md`). Each ADR includes: **Context**, **Decision**,
**Alternatives**, **Consequences**, **Risks**, **Review conditions**, and a status
(`proposed` / `accepted` / `superseded`).

## Index

| # | Title | Status |
|---|---|---|
| [0001](0001-orchestrator-as-cross-platform-source-of-truth.md) | Orchestrator as cross-platform source of truth | accepted |
