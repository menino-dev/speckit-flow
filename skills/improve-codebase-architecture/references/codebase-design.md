# Codebase Design Vocabulary & Heuristics

Reference vocabulary for `/improve-codebase-architecture` (adapted for Antigravity + Spec Kit). Use these precise concepts instead of vague labels like "cleaner code" or "better separation".

## Core Concepts

- **Deep Module**: Hides significant logic, state coordination, or policy behind a small, stable, easy-to-call interface. Callers do not need to understand internal steps to use it safely.
- **Shallow Module**: Exposes an interface nearly as large or complex as the implementation beneath it (pass-through wrappers, fragmented 10-line helpers that force callers to stitch steps together, or excessive config flags mirroring internals).
- **Seam**: A clean boundary where behavior can be understood, replaced, or tested in isolation without mocking five internal collaborators.
- **Locality**: Keeping related logic, invariants, and state transitions together so a developer (or AI agent) can read one place and understand the full behavior instead of jumping across 4–6 files.
- **Leverage**: A structural change that simplifies multiple callers, eliminates a whole class of bugs, or collapses brittle mocked unit tests into a few confident boundary tests.
- **Adapter**: A thin boundary layer isolating an external system (HTTP API, database, filesystem, third-party SDK) so core domain modules stay deep and testable.

## The Three Shallowness Patterns to Hunt

1. **Test-Induced Fragmentation (Low Locality)**: Pure helpers extracted solely to make unit tests easy, while real bugs live in how callers orchestrate them.
2. **Leaky Seams**: Callers must pass internal flags, respect undocumented call order, or know internal data shapes to use a module.
3. **Scattered Concepts**: Understanding or changing one domain concept requires opening 4–5+ files with parallel edits.

## The Two Mandatory Filters

1. **The Deletion Test**: Ask *"If we collapse or deepen this boundary, does complexity genuinely move behind a smaller, calmer interface — or does it just spread across callers?"* Only keep candidates where total interface surface shrinks and caller simplicity rises.
2. **Living Code Filter (Git Churn)**: Unless scoped to a specific `spec.md` or directory, inspect recent commits (`git log --name-only -n 30`) and focus on files that actually change. Deepening untouched, stable code rarely pays back.

## Strength Badges

| Badge | Criterion | Action |
|---|---|---|
| **Strong** | Clearly passes the deletion test, hits active/affected code, delivers measurable locality or testing leverage. | Worth acting on now (or as a Preparatory Phase in `plan.md`). |
| **Worth exploring** | Plausible deepening, but payoff depends on upcoming roadmap or feature direction. | Optional; consider only if aligned with current feature. |
| **Speculative** | Marginal gain or uncertain tradeoff. | Ignore by default. If **all** findings are `Speculative`, report that the codebase architecture is healthy. |

## Constitutional Guardrail (YAGNI & Governance)

Always read `.specify/memory/constitution.md` before scoring candidates:
- Never propose abstractions, layers, or patterns forbidden by the project's Constitution.
- A candidate that *adds* indirection without shrinking caller complexity fails both YAGNI and the Deletion Test.
- When a user permanently rejects a candidate, offer to record a concise ADR in `docs/adr/NNN-<slug>.md` so future runs skip it automatically.
