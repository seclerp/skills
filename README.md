# Skills

A collection of personal, cross-agent skills designed to enhance productivity and streamline developer workflows across **Claude Code**, **Codex CLI**, and **Junie**.

## Structure

```text
skills/
├── kotlin/
│   └── kt-add-test/       # Converts Kotlin snippets/issues into compiler testData tests
│       ├── SKILL.md       # Core classification & execution workflow
│       └── reference/     # Directives and testData directory mappings
└── prompting/
    └── create-prompt/     # Prepares repo-grounded prompts for other agents
        ├── SKILL.md       # Workflow, hard rules, and prompt architecture
        └── reference/     # Templates and pre-flight verification checklist
```

## Catalog

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
  - **Purpose:** Prepares a comprehensive, highly-structured, and deeply codebase-grounded prompt for another agent (planning agent, coding agent, subagent, or external model) or human developer to plan or implement a technical task.
  - **Key Features:**
    - **Read-Only Reconnaissance:** Forbids modifying code or entering self-directed execution planning; investigates the codebase to ground the prompt in exact relative file paths, symbols, and precedents.
    - **Test Impact Assessment:** Actively searches existing test suites (`testData`, unit tests, box tests) to identify existing tests that will break and outline necessary updates or suppressions.
    - **Role-Tailored Architecture:** Adapts prompt structure to the recipient (planning agent architecture briefs vs coding agent execution instructions).
    - **Zero-Clutter Delivery:** Emits only the ready-to-run prompt or writes cleanly to a file (e.g. `prompt.md`), eliminating conversational preamble and meta-recaps.

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
