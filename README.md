# Skills

A collection of personal, cross-agent skills designed to enhance productivity and streamline developer workflows across **Claude Code**, **Codex CLI**, and **Junie**.

## Structure

```text
skills/
├── git/
│   └── commit-changes/    # Splits local changes into approved, convention-following commits
│       └── SKILL.md       # Analysis, convention discovery, approval gate, commit workflow
├── kotlin/
│   └── kt-add-test/       # Converts Kotlin snippets/issues into compiler testData tests
│       ├── SKILL.md       # Core classification & execution workflow
│       └── reference/     # Directives and testData directory mappings
└── prompting/
    └── create-prompt/     # Turns raw thoughts into WHAT-only briefs for planning agents
        ├── SKILL.md       # Workflow, hard rules, and brief architecture
        └── reference/     # WHAT-brief templates and WHAT-ness checklist
```

## Catalog

### Git

- **[`git/commit-changes`](./git/commit-changes)**
  - **Purpose:** Turns the current working tree into a clean, reviewable series of commits: analyzes local changes, proposes a logically split commit list, waits for explicit approval, then commits.
  - **Key Features:**
    - **Convention Discovery:** Follows user instructions, repo guidelines (`AGENTS.md`, `CONTRIBUTING.md`, …), commit tooling config and `git log` style; falls back to one-line Conventional Commits without co-authorship or trailers.
    - **Logical Grouping:** One logical change per commit, each buildable on its own; lockfile hunks follow the manifest changes that caused them; refactors → features/fixes → docs.
    - **Approval Gate:** Nothing is committed until the user approves unconditionally; questions, concerns or "ok, but…" trigger a revised full proposal instead.
    - **Non-Destructive Splits:** Splits files across commits at the index level only, never touching working files; no amend, rebase, push or `--no-verify` unless asked.

### Kotlin

- **[`kotlin/kt-add-test`](./kotlin/kt-add-test)**
  - **Purpose:** Converts an arbitrary Kotlin code snippet (or a `KT-XXXXX` YouTrack issue ID) into a correctly-placed, correctly-annotated compiler test file under `compiler/testData/` within any checkout or worktree of the Kotlin compiler repository.
  - **Key Features:**
    - **Intelligent Classification:** Distinguishes between `codegen/box`, JVM-specific `codegen/boxJvm`, `diagnostics/tests`, `ir/irText`, and bytecode tests, enforcing proper placement rules (such as redirecting away from `js/js.translator/testData`).
    - **Directive Inference:** Automatically infers and injects required header directives (`WITH_STDLIB`, `WITH_REFLECT`, `WITH_COROUTINES`, `TARGET_BACKEND`, `RUN_PIPELINE_TILL`, `FILE`/`MODULE`, etc.).
    - **YouTrack Integration:** Can fetch issue details and derive reproduction snippets directly from YouTrack when only an issue ID is provided.
    - **Safety-First Gradle Gate:** Stops and requests explicit user confirmation before running `./gradlew generateTests` or executing targeted tests.

### Prompting

- **[`prompting/create-prompt`](./prompting/create-prompt)**
  - **Purpose:** Turns unstructured thoughts into a structured, self-contained statement of **what** should be done — goal, background, desired outcome, acceptance criteria, scope, constraints, and open questions — for a downstream planning agent to turn into a **how**.
  - **Pipeline Position:** `raw thoughts → create-prompt → WHAT brief → planning agent → plan → review → implementation → review`.
  - **Key Features:**
    - **No Solution Prescription:** Hard rule against step-by-step logic, signatures, file-edit instructions, and build/test commands; user-mandated solution details are demoted to labelled constraints.
    - **Ask, Don't Invent:** Ambiguity is either raised as focused questions or surfaced under *Open Questions for the Planner*, never silently guessed; false premises in the user's input are corrected.
    - **Orientation Recon (read-only):** Grounds subsystem names, entry points, current behaviour, and a verifiability surface — vocabulary, not design; prior art is reference-only.
    - **WHAT-ness Gates:** Rejecting checklist — no prescription, design space left open, observable acceptance criteria, unknowns surfaced — plus a ~40–80 line target to stop volume dragging the brief into HOW.
    - **Zero-Clutter Delivery:** Emits only the brief, inline or written cleanly to a file (e.g. `prompt.md`).

## Installation & Setup

To maximize efficiency and maintain a single source of truth across agents, install skills by creating directory symlinks from your personal agent skill directories to the checked-out skills in this repository.

### Installing `kt-add-test`

```bash
# Canonical location (e.g. for Claude Code)
mkdir -p ~/.claude/skills
ln -sfn ~/Pets/skills/kotlin/kt-add-test ~/.claude/skills/kt-add-test

# Codex CLI
mkdir -p ~/.codex/skills
ln -sfn ~/.claude/skills/kt-add-test ~/.codex/skills/kt-add-test

# Junie
mkdir -p ~/.junie/skills
ln -sfn ~/.claude/skills/kt-add-test ~/.junie/skills/kt-add-test
```

### Installing `commit-changes`

```bash
# Canonical location (e.g. for Claude Code)
mkdir -p ~/.claude/skills
ln -sfn ~/Pets/skills/git/commit-changes ~/.claude/skills/commit-changes

# Codex CLI
mkdir -p ~/.codex/skills
ln -sfn ~/.claude/skills/commit-changes ~/.codex/skills/commit-changes

# Junie
mkdir -p ~/.junie/skills
ln -sfn ~/.claude/skills/commit-changes ~/.junie/skills/commit-changes
```

### Installing `create-prompt`

```bash
# Canonical location (e.g. for Claude Code)
mkdir -p ~/.claude/skills
ln -sfn ~/Pets/skills/prompting/create-prompt ~/.claude/skills/create-prompt

# Codex CLI
mkdir -p ~/.codex/skills
ln -sfn ~/.claude/skills/create-prompt ~/.codex/skills/create-prompt

# Junie
mkdir -p ~/.junie/skills
ln -sfn ~/.claude/skills/create-prompt ~/.junie/skills/create-prompt
```

Symlinking the directory ensures any improvements or updates are immediately available across all agents and worktrees without manual synchronization.
