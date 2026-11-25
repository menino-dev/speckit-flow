---
name: improve-codebase-architecture
description: >-
  Survey an existing codebase for architectural deepening opportunities (shallow
  modules -> deep modules, missing seams, low locality) without modifying source
  code. Produces a visual report with Mermaid before/after diagrams and ranked
  strength badges, then guides a focused design decision on the chosen candidate.
  Runs standalone via /improve-codebase-architecture or inside /speckit-flow
  (as a spec-guided pre-plan check or full architecture audit entry).
---

# Improve Codebase Architecture

Survey an existing codebase for **deepening opportunities** — places where shallow, leaky, or scattered modules can be consolidated into **deep modules** with smaller interfaces, higher locality, and simpler tests.

> **Hard Rule — Zero Source Code Edits**: This skill is strictly a **survey and design-decision** tool. It NEVER edits application source code. Actual refactoring happens afterwards through the normal Spec Kit flow (`specify -> clarify -> plan -> tasks -> analyze -> implement`).

Load [references/codebase-design.md](./references/codebase-design.md) when evaluating candidates to apply the exact vocabulary (*deep module*, *shallow module*, *seam*, *locality*, *leverage*, *deletion test*) and strength badges (`Strong`, `Worth exploring`, `Speculative`).

---

## Determine Execution Mode

Choose the mode automatically from context:

1. **`spec-guided` (Pre-Plan Check inside `speckit-flow`)**:
   - Triggered when an active `spec.md` already exists and the user is adding a feature to an **existing (brownfield)** codebase.
   - **Skip immediately** if the workspace is greenfield (no existing application source code yet).
   - Scopes the scan strictly to the modules/files the active `spec.md` will touch (plus their direct callers and recent commits).
   - Core question: *"How can we make this upcoming change easy by deepening a shallow module or adding a missing seam first?"*

2. **`full-audit` (Standalone `/improve-codebase-architecture` or Audit Entry in `speckit-flow`)**:
   - Triggered when invoked directly via `/improve-codebase-architecture` or when the user asks `/speckit-flow` to audit/improve codebase architecture.
   - Unless the user points to a specific directory or spec, starts from recent commit history (`git log --name-only -n 30`) to focus strictly on actively changing areas.

---

## Execution Protocol

### Step 1: Load Domain & Governance Context (Read-Only)

In parallel, read (if they exist):
- `.specify/memory/constitution.md` — non-negotiable architectural & YAGNI constraints.
- `GLOSSARY.md` — canonical domain vocabulary (use real domain nouns in candidate titles, never generic class names).
- `docs/adr/` — previously accepted/rejected Architecture Decision Records (never re-propose a previously rejected candidate).
- Active `spec.md` (in `spec-guided` mode).

### Step 2: Explore Living Code & Hunt for Shallowness

- In `full-audit` mode (without an explicit target path), run `git log --name-only --pretty=format: -n 30` to identify the highest-churn source directories/files.
- Use parallel file reads (or delegate broad exploration to the native `research` subagent in large repositories) to inspect how callers interact with target modules and their tests.
- Look specifically for the 3 shallowness patterns from [references/codebase-design.md](./references/codebase-design.md):
  1. Test-induced fragmentation (low *locality*).
  2. Leaky *seams* exposing internals to callers.
  3. Domain concepts scattered across 4–5+ files.
- Filter **every** candidate through:
  1. **The Deletion Test**: complexity must shrink behind a smaller, calmer interface — never spread onto callers.
  2. **Constitution Check**: reject any proposal that violates `.specify/memory/constitution.md` (especially YAGNI / stack constraints).
  3. **ADR Check**: exclude ideas already rejected in `docs/adr/`.

### Step 3: Score & Decide Whether to Surface

Classify each surviving candidate with a badge:
- **`Strong`**: Clearly passes the deletion test, hits active/affected code, yields clear locality or test-simplification leverage.
- **`Worth exploring`**: Plausible deepening, payoff depends on roadmap.
- **`Speculative`**: Marginal or uncertain gain.

**Fast-Path Exit (No-Bloat Rule)**:
- In **`spec-guided` mode**: If **zero `Strong` candidates** exist (only `Speculative` or none), do **NOT** interrupt the user or write a full report. Record a concise 1–2 line summary ("Architecture pre-check: no preparatory deepening required") for `research.md` and hand control immediately back to `/speckit-plan`.
- In **`full-audit` mode**: If **all** candidates are `Speculative` (or none pass the deletion test), state honestly that the codebase has no high-leverage deepening candidates right now — never invent fake refactor work.

### Step 4: Generate the Visual Architecture Report

When presenting candidates (`full-audit` mode, or `spec-guided` mode with `Strong` findings):

- **Default Format (Antigravity Native — Zero CDN Risk)**:
  - Write a clean Markdown artifact `<artifactDir>/architecture-review-<YYYYMMDD-HHMM>.md` (with `UserFacing: true`, `RequestFeedback: false`) using native **Mermaid** (`flowchart LR` / `flowchart TD`) before/after diagrams and clickable `file:///` links.
  - *(If the user explicitly asks for an HTML file, generate a self-contained HTML file with **inline CSS and hand-built inline SVG diagrams** — never rely on external CDN scripts that break offline.)*
- **Required Content per Candidate Card**:
  1. **Domain-oriented title** + **Strength Badge** (`Strong` / `Worth exploring` / `Speculative`).
  2. **Files involved** (clickable links).
  3. **The friction** (what is shallow, leaky, or scattered right now).
  4. **Plain-English deepening proposal** (what moves behind the seam and what the smaller interface looks like).
  5. **Concrete payoff**: express strictly in terms of **locality**, **leverage**, and **which tests become simpler** (never vague "it's cleaner").
  6. **Before / After diagram** (Mermaid flowchart with quoted labels for special characters).
- End the report with a single **Top Recommendation** stating which 1 candidate to tackle first and why.

### Step 5: Stop & Ask (Never Auto-Grill Before Selection)

**STOP** after presenting the report. Do not launch a long interview on your own, and do not modify source code.

- **In `spec-guided` mode** (inside `speckit-flow`, right before `/speckit-plan`):
  - Keep it lightweight (avoid duplicating `/speckit-clarify`). Present the top `Strong` candidate(s) briefly and use `ask_question` with 3 clear choices:
    1. **(Recommended)** Include this deepening as a **Preparatory Phase** inside the current `plan.md` / `tasks.md` before implementing the feature.
    2. Keep the feature plan focused as-is and save this deepening as a separate follow-up backlog item.
    3. Reject this deepening permanently and record a short ADR in `docs/adr/` so future runs ignore it.

- **In `full-audit` mode**:
  - Ask the user which **ONE** candidate they want to explore (enforce **1 candidate per session** so context stays sharp; offer to note any remaining strong candidates as future follow-ups).
  - If the user rejects a candidate for a lasting architectural reason, offer to record a concise ADR in `docs/adr/NNN-<slug>.md` (only create it if they confirm).
  - Once the user picks **1 candidate**:
    - **If running inside `speckit-flow`**: pass the selected candidate directly into **`/speckit-specify` → `/speckit-clarify`** (where `/speckit-clarify` acts as the focused architectural grilling on interface shape, seam boundaries, constraints, and surviving tests) → `/speckit-plan` → `/speckit-tasks` → `/speckit-analyze` → `/speckit-implement`.
    - **If running standalone**: run a concise, capped grilling loop (3–5 targeted questions via `ask_question`, covering constraints, what lives behind the seam, interface signature, and test strategy), optionally update/create `GLOSSARY.md` if domain terms were clarified (with user agreement), and output a crisp **Design Decision Summary** ready to feed into `/speckit-flow` or `/speckit-specify`.
