---
name: speckit-flow
description: >-
  Orchestrate the complete Spec-Driven Development (SDD) workflow with Spec Kit for
  starting new projects, adding features, or running architecture deepening / web
  modernization refactors. Trigger when the user invokes /speckit-flow or asks to
  start a project, add a feature, or audit/improve architecture following the Spec Kit
  pipeline: constitution (when missing) -> [architecture/modernization audit if
  requested] -> specify -> clarify (always asking user questions) -> [spec-guided
  architecture check on brownfield] -> plan (with Modern Web Guidance on Web UI tasks)
  -> tasks -> analyze (asking confirmation before any constitution edits) -> implement.
---

# SpecKit Flow — Full Spec-Driven Development Pipeline

Execute Spec-Driven Development from a single user prompt describing a **new project**, a **new feature**, or an **architecture improvement / web modernization audit**. Follow every phase in exact order, respecting all interactive gates. Communicate in the same language as the user's prompt.

## Automatic Mode Routing

Detect the user's intent and repository state during Pre-Flight:

1. **New Project (Greenfield — empty/no app source code yet)**:
   `Pre-Flight → [Phase 0: /speckit-constitution] → Phase 1: /speckit-specify → Phase 2: /speckit-clarify → Phase 3: /speckit-plan → Phase 4: /speckit-tasks → Phase 5: /speckit-analyze → Phase 6: /speckit-implement`
   *(Skips architecture review automatically — no existing modules to deepen yet.)*

2. **Feature Addition (Brownfield — existing codebase with source code)**:
   `Pre-Flight → [Phase 0: /speckit-constitution if undefined] → Phase 1: /speckit-specify → Phase 2: /speckit-clarify → Phase 2.5: Spec-Guided Architecture Check → Phase 3: /speckit-plan → Phase 4: /speckit-tasks → Phase 5: /speckit-analyze → Phase 6: /speckit-implement`
   *(Runs a fast, spec-guided deepening check on affected modules before planning: "make the change easy, then make the easy change".)*

3. **Architecture Audit / Web Modernization Mode** *(when user asks to audit/improve architecture or broadly modernize legacy web code instead of describing a specific feature)*:
   `Pre-Flight → [Phase 0: /speckit-constitution if undefined] → Phase 0.5: Full Architecture Audit (+ optional modern-web-guidance legacy-scan for open-ended web modernization) → User picks 1 candidate / modernization scope → Phase 1: /speckit-specify → Phase 2: /speckit-clarify (acts as design grilling) → Phase 3: /speckit-plan → Phase 4: /speckit-tasks → Phase 5: /speckit-analyze → Phase 6: /speckit-implement`

Use `update_steps` immediately after Pre-Flight to show the exact active phase checklist to the user, and update it as each phase finishes.

---

## How to Load Dependent Skills

Before running any phase `<step>` (`constitution`, `specify`, `clarify`, `plan`, `tasks`, `analyze`, `implement`), locate and read its `SKILL.md` in this resolution order:

1. Workspace `.agents/skills/speckit-<step>/SKILL.md`
2. Workspace `.github/skills/speckit-<step>/SKILL.md`
3. Machine fallback `~/.gemini/antigravity/skills/speckit-<step>/SKILL.md` or `~/.gemini/config/skills/speckit-<step>/SKILL.md`

For companion skills:
- **Architecture Review** (Phases `0.5` and `2.5`): read the bundled sibling skill `../improve-codebase-architecture/SKILL.md` (and its `references/codebase-design.md`).
- **Modern Web Guidance** (Phases `0`, `0.5`, `3`, `6` on Web UI tasks): read the installed `modern-web-guidance` skill (typically under `~/.gemini/config/plugins/modern-web-guidance-plugin/skills/modern-web-guidance/SKILL.md` or available in active skills). For Chrome Extensions, read the installed `chrome-extensions` skill.

Follow the instructions inside each loaded `SKILL.md` (including `.specify/scripts/` helpers and any `.specify/extensions.yml` hooks), while strictly enforcing the flow gates below.

---

## Step-by-Step Execution Protocol

### Pre-Flight: Workspace, Scope & Constitution Detection

Run one read-only check at the repository root:

1. **Check `.specify/`**:
   - If `.specify/` does **not** exist:
     - Check whether `specify` CLI is installed (`Get-Command specify` on Windows or `which specify` on Unix).
     - If available, initialize Spec Kit non-interactively in the current directory:
       - Windows: `specify init --here --force --non-interactive --integration copilot --script ps`
       - Linux/macOS: `specify init --here --force --non-interactive --integration copilot --script sh`
     - Inform the user that Spec Kit was initialized in the workspace.
     - If `specify` CLI is not installed, create `.specify/memory/` and `.specify/templates/` cleanly or instruct the user to install `specify-cli` before proceeding.
2. **Check `.specify/memory/constitution.md`**:
   - Read `.specify/memory/constitution.md` if it exists.
   - Classify the constitution as **UNDEFINED** if:
     - The file does not exist, OR
     - The file is empty, OR
     - The file still contains unfilled scaffold placeholders matching `\[[A-Z0-9_]+\]` such as `[PROJECT_NAME]`, `[PRINCIPLE_1_NAME]`, `[CONSTITUTION_VERSION]`, or `[RATIFICATION_DATE]`.
   - Classify the constitution as **DEFINED** otherwise.
3. **Detect Greenfield vs. Brownfield**:
   - Check whether application source code already exists (e.g., `src/`, `app/`, `lib/`, `packages/`, or tracked source files).
   - No source files yet → **Greenfield** (skip Phase 2.5).
   - Existing source code → **Brownfield** (enable Phase 2.5 on feature additions, or Phase 0.5 on architecture/modernization audits).
4. **Detect Web UI / Chrome Extension Scope (`web_guidance` / `extension_guidance`)**:
   - Set **`web_guidance = true`** if the task or workspace involves HTML, CSS, DOM, browser APIs, or client-side JS UI (modals/dialogs/popovers, tables, forms, validation, autofill, dark mode, animations/scroll effects, container queries, anchor positioning, view transitions, accessibility, Core Web Vitals, images, dates/timezones, sanitization, passkeys, React/Vite/Next/Vue/Astro/vanilla web UI).
   - Set **`extension_guidance = true`** if the task involves Chrome/browser extensions (`manifest.json`, Manifest V3, service worker, content scripts, side panel, Chrome Web Store).
   - Set **both `false`** for pure backend (SQL, ORMs, API routes), CLI, infra/scripts (CI/CD, Docker, Python/Go, Git), or trivial tooling/copy tasks — completely skipping `modern-web-guidance` with zero wasted tokens.

---

### Phase 0: `/speckit-constitution` *(ONLY when constitution is UNDEFINED)*

1. Load and follow `speckit-constitution/SKILL.md`.
2. Inspect the workspace (`package.json`, `pyproject.toml`, `README.md`, existing files, if any) AND the user's initial prompt to infer candidate project principles (stack, architecture, testing, design/UX, simplicity, quality gates).
3. If `web_guidance = true`, include a clear **Browser Support / Baseline Policy** rule in the constitution (defaulting to *Baseline Widely Available* without extra polyfills, or confirming a custom target such as *Baseline YYYY* or *Newly Available with lightweight fallback*).
4. If the project is brand-new and key architectural principles are not yet specified in the user's prompt, ask 2–4 concise questions (using `ask_question` when discrete choices fit) so the constitution truly reflects the user's standards.
5. Draft and write `.specify/memory/constitution.md` at `v1.0.0` (with today's ISO date and zero unexplained placeholders).
6. Briefly present the established principles and immediately proceed to the next phase (Phase 0.5 for architecture/modernization audits, or Phase 1 for projects/features).

---

### Phase 0.5: Full Architecture Audit / Web Modernization *(ONLY in Audit / Modernization Mode)*

1. **Architecture Audit**: Load and follow `../improve-codebase-architecture/SKILL.md` in **`full-audit` mode**:
   - Survey recent commit hotspots + `constitution.md` + `GLOSSARY.md` + `docs/adr/`, apply the Deletion Test, and produce the visual Markdown report artifact with Mermaid before/after diagrams and strength badges (`Strong`, `Worth exploring`, `Speculative`).
2. **Open-Ended Web Modernization** *(ONLY when `web_guidance = true` AND the user explicitly requested a broad, open-ended web modernization / legacy scan with no specific target feature)*:
   - Load the installed `modern-web-guidance` skill and run its **Alternative flow (`legacy-scan <path> --json`)**, verifying every finding against its `keepWhen`/`dropWhen` rubric to filter out false positives (fallbacks, vendor code, lookalikes).
   - **Never** run `legacy-scan` when the user asked to add, update, fix, or audit a specific feature or component.
3. **STOP** and ask the user to choose **1 candidate** (or approved modernization scope) to take into Spec Kit (or confirm no refactor is needed / record an ADR for rejected candidates).
4. Once selected, pass that candidate directly into **Phase 1: `/speckit-specify`**.

---

### Phase 1: `/speckit-specify`

1. Load and follow `speckit-specify/SKILL.md`.
2. Using the user's project/feature description (or the selected candidate from Phase 0.5):
   - Determine the feature directory (`specs/<NNN>-<short-name>/`), create it along with `checklists/`, and persist `{"feature_directory": "specs/<NNN>-<short-name>"}` to `.specify/feature.json`.
   - Author `spec.md` (prioritized User Stories with acceptance scenarios, Edge Cases, testable Functional Requirements, Key Entities, measurable Success Criteria, Assumptions).
   - Author and validate `checklists/requirements.md`.
3. **Do NOT write application code** in this phase.
4. Summarize the generated `spec.md` concisely (with clickable `file://` link) and **immediately enter Phase 2: `/speckit-clarify`**.

---

### Phase 2: `/speckit-clarify` *(MANDATORY INTERACTIVE GATE)*

1. Load and follow `speckit-clarify/SKILL.md` (run `check-prerequisites.ps1 -Json -PathsOnly` / `.sh`, load `constitution.md` and `spec.md`).
2. **ALWAYS ask the user questions about the feature/project** so no requirement, edge case, UX detail, integration behavior, or acceptance boundary is left assumed or underspecified:
   - Scan all ambiguity taxonomy areas: Functional Scope & Behavior, Domain & Data Model, Interaction & UX Flow, Non-Functional Attributes (performance, security, accessibility, responsiveness), Integrations, Edge Cases & Failure Handling, and Completion Signals.
   - *(If coming from Phase 0.5 Architecture Audit, focus questions as the architectural grilling loop: constraints, what moves behind the seam, target interface shape, and which tests survive/simplify.)*
   - Formulate **3 to 5 high-impact, concrete questions** (minimum 2 even if the prompt seemed thorough — probe edge cases, error/empty states, scope boundaries, or acceptance tradeoffs).
   - Present questions clearly with a **Recommended** option and brief rationale ("why it matters") using `ask_question` (or direct questions if open-ended). Wait for the user's answers before moving on.
   - **Never silently skip this phase.** Only bypass if the user explicitly instructs you to skip clarification.
3. After receiving the user's answers:
   - Integrate every accepted answer into `spec.md` (`## Clarifications` → `### Session YYYY-MM-DD` and target sections, removing contradicted vague text).
   - Re-validate `checklists/requirements.md` against the updated `spec.md`.
4. Proceed to **Phase 2.5** (brownfield feature addition) or directly to **Phase 3: `/speckit-plan`** (greenfield or already coming from Phase 0.5).

---

### Phase 2.5: Spec-Guided Architecture Check *(Brownfield Feature Additions Only)*

1. Skip automatically if the repo is **greenfield** OR if this run already started from **Phase 0.5**.
2. Load and follow `../improve-codebase-architecture/SKILL.md` in **`spec-guided` mode**:
   - Inspect only the existing modules that the finalized `spec.md` will touch (plus their direct callers and recent commits), respecting `.specify/memory/constitution.md`, `GLOSSARY.md`, and `docs/adr/`.
   - Apply the Deletion Test strictly to detect shallow modules or missing seams that would make implementing or testing the feature painful.
3. **Fast-Path Decision**:
   - **Zero `Strong` candidates** (none or only `Speculative`) → Do **NOT** stall the user. Record a 1-line note ("Spec-guided architecture check: no preparatory deepening needed") to include in `research.md`, and proceed immediately to **Phase 3: `/speckit-plan`**.
   - **One or more `Strong` candidates** → Present the compact card(s) with Mermaid before/after diagram and ask **1 quick `ask_question`** (never a second long grilling):
     - **(Recommended)** Include as a **Preparatory Phase** in `plan.md` & `tasks.md` before building the feature.
     - Save as a separate future backlog item and proceed with the feature plan as-is.
     - Reject permanently and record a brief ADR in `docs/adr/`.
4. Carry the user's decision into **Phase 3: `/speckit-plan`**.

---

### Phase 3: `/speckit-plan` *(Includes Modern Web Guidance Research on Web UI Tasks)*

1. Load and follow `speckit-plan/SKILL.md`.
2. Run `.specify/scripts/powershell/setup-plan.ps1 -Json` (or `.sh` equivalent).
3. **Modern Web Guidance Lookup (`web_guidance = true` or `extension_guidance = true`) — MANDATORY BEFORE selecting packages or designing UI/DOM architecture**:
   - Read and follow the installed `modern-web-guidance` skill (and `chrome-extensions` skill when `extension_guidance = true`).
   - For every distinct web UI/platform capability required by `spec.md` (e.g., modal/dialog, popover, table, dark mode, scroll animation, forms/validation, anchor positioning, container queries, accessibility, CWV, dates/timezones, sanitization), run focused 2–5 word generic English `search` queries and `retrieve` the matching guides (plus relevant discipline guides like `accessibility`, `forms`, `css`, `performance`, `security`).
   - Prefer native modern web platform capabilities backed by the retrieved guides over adding third-party libraries or hand-rolled legacy patterns (aligned with YAGNI and Constitution).
   - Apply the project's **Browser Support / Baseline Policy** from `.specify/memory/constitution.md` (default: Baseline Widely Available without fallback; follow guide fallbacks for non-Baseline features).
   - Record all retrieved `guide-id`s, chosen modern APIs, and required fallbacks in **`research.md`** (and reflect them in `plan.md` & `contracts/`).
   - *Graceful degradation*: if `modern-web-guidance` is disabled or offline, try `npx --offline` or note it briefly in `research.md` and proceed without stalling.
4. Evaluate every principle in `.specify/memory/constitution.md` inside the Constitution Check table.
   - If a requirement appears to require a constitution amendment, note it clearly in `plan.md` (do **not** silently alter `constitution.md` yet — that confirmation happens explicitly at the Phase 5 gate).
5. Generate Phase 0 & Phase 1 design artifacts inside the active feature directory:
   - `plan.md` (including any approved Preparatory Deepening phase from Phase 2.5 and Modern Web Guidance architecture choices)
   - `research.md` (recording architecture check + retrieved `modern-web-guidance` guide IDs and decisions)
   - `data-model.md`
   - `contracts/` (when interfaces/APIs/components apply)
   - `quickstart.md`
6. Immediately proceed to **Phase 4: `/speckit-tasks`**.

---

### Phase 4: `/speckit-tasks`

1. Load and follow `speckit-tasks/SKILL.md`.
2. Run `.specify/scripts/powershell/setup-tasks.ps1 -Json` (or `.sh` equivalent).
3. Generate `tasks.md` organized by phase (Setup → Foundational [including any approved Preparatory Deepening tasks] → User Stories in priority order → Polish) with strict checklist format:
   `- [ ] T001 [P?] [US?] Clear action with exact file path`
4. Verify 100% requirement coverage and clear dependencies.
5. Immediately proceed to **Phase 5: `/speckit-analyze`**.

---

### Phase 5: `/speckit-analyze` *(MANDATORY CONSTITUTION & QUALITY GATE)*

1. Load and follow `speckit-analyze/SKILL.md` (**strictly read-only during analysis**).
2. Cross-check `spec.md`, `plan.md`, `tasks.md`, and `.specify/memory/constitution.md` for:
   - Duplications, ambiguities, underspecification, coverage gaps, and cross-artifact inconsistencies.
   - **Constitution Alignment**: Check whether the planned feature/project conflicts with any MUST rule in `.specify/memory/constitution.md` OR introduces a new architectural/quality pattern (e.g., browser support policy, accessibility bar, UI standard) that should be formalized in the constitution.
3. Output the compact **Specification Analysis Report** (findings table, requirement coverage table, metrics).
4. **Decision Gate before Phase 6**:
   - **Branch A — Constitution change IS needed**:
     1. **STOP before modifying `.specify/memory/constitution.md` and before starting `/speckit-implement`.**
     2. Present the exact proposed changes to `.specify/memory/constitution.md`:
        - Which principle(s)/section(s) will be added or edited.
        - Before vs. After (or exact diff/new text).
        - Semantic version bump (`MAJOR`, `MINOR`, or `PATCH`) and concise rationale.
     3. Ask the user for **explicit confirmation** to apply the constitution amendment (and any related critical fixes) and continue to `/speckit-implement`.
     4. Wait for the user's response. Once confirmed:
        - Run `/speckit-constitution` to apply the approved amendment to `.specify/memory/constitution.md`.
        - Re-align `plan.md`/`tasks.md` if affected, then proceed to Phase 6.
   - **Branch B — No constitution change needed, but `CRITICAL` or `HIGH` findings exist**:
     - Present the findings and concrete remediation edits, ask the user for confirmation to fix them before implementing, and wait for approval.
   - **Branch C — No constitution change needed AND zero `CRITICAL`/`HIGH` findings**:
     - State clearly that `.specify/memory/constitution.md` is 100% aligned (no amendment needed) and all quality gates pass.
     - Proceed directly into **Phase 6: `/speckit-implement`** (unless the user asked to pause before coding).

---

### Phase 6: `/speckit-implement`

1. Load and follow `speckit-implement/SKILL.md`.
2. Verify `checklists/requirements.md` status (all items passing).
3. Execute `tasks.md` phase by phase, respecting dependencies and parallelizing independent file edits where appropriate.
4. **Modern Web Guidance Compliance Check (`web_guidance = true`)**:
   - Cross-check all created/edited HTML, CSS, and client-side JS against the guides retrieved in Phase 3 (re-retrieving if context was compacted — never quote guide IDs from memory).
   - Apply framework adaptation rules cleanly (e.g., React/Vite ref calls for `showModal()`, `useId()` for unique IDs/`anchor-name`, top-level `@starting-style`, SSR guards, and guide-mandated fallbacks for non-Baseline features).
5. Mark every completed task `[x]` in `tasks.md` as you complete each phase.
6. Run the project's build, type-check, lint, and test commands (plus `quickstart.md` verification steps) to prove the implementation works end-to-end.
7. **Git Hygiene**: Do **not** run `git commit` or `git push` unless the user explicitly asks for it.
8. Mark all steps done via `update_steps` and deliver a concise Completion Report linking all generated Spec Kit artifacts, modified source files, and any applied `modern-web-guidance` guide IDs.
