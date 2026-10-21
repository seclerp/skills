---
name: commit-changes
description: >
  Analyzes local git changes, proposes a logically split list of commits that follow the repository's
  commit conventions (falling back to Conventional Commits one-liners), never adds co-authorship trailers
  even if the repository allows them (unless the user explicitly asks), waits for
  explicit user approval, applies requested adjustments, and only then creates the commits. Use whenever
  the user asks to commit local changes, split work into commits, or "come up with a commit list".
user-invocable: true
---

# commit-changes

Turns the current working tree into a clean, reviewable series of commits. The skill has a hard
**approval gate**: nothing is staged-and-committed until the user explicitly approves the proposed list.

## Hard Rules

1. **Never commit before explicit approval.** Approval means an unambiguous "yes / approved / go / commit"
   with no open concerns, questions or change requests in the same message. Anything else (a question,
   "looks good but…", a requested tweak, silence) is **not** approval — address it, re-propose, and ask again.
2. **Never commit files the user excluded.** Leave them exactly as they are on disk. If they were already
   staged, unstage them (`git restore --staged <file>`), never delete or revert them.
3. **Never modify the user's working files** to make a split work. Split at the index level only (see
   "Line-level splits").
4. **No history rewriting** (`--amend`, `rebase`, `reset --hard`, force-push) and **no push** unless the
   user explicitly asks.
5. **No co-authorship, ever, by default.** Never add `Co-authored-by` trailers (or any other mention of
   an AI/agent/tool as co-author, e.g. "Generated with …", `--trailer "Co-authored-by: …"`) — **even if
   the repository guidelines, commit template, tooling config or `git log` history use or allow them**.
   This overrides every convention found in Step 2. The only exception is an explicit request from the
   user in the current conversation to add co-authorship. Other trailers (e.g. `Signed-off-by`, ticket
   refs) follow the repository rules.
6. Do not run hooks-bypassing flags (`--no-verify`) unless the user asks. If a hook fails, report it and
   fix the cause or ask.

## Workflow

### Step 1 — Analyze local changes

Collect the full picture before proposing anything:

```bash
git status --short
git diff --stat; git diff --cached --stat       # unstaged vs. already staged
git diff -M HEAD --stat                          # renames detected
git log --oneline -15                            # existing message style
```

- Read the actual diffs (`git diff -M HEAD -- <paths>`), including untracked files (open them).
- For lockfiles / generated files (`Cargo.lock`, `package-lock.json`, `gradle.lockfile`, …) map each hunk
  to the manifest change that caused it, so it can go into the matching commit.
- Note files that are clearly scratch/local artifacts (reports, notes, IDE output, secrets, `.env`) and
  propose excluding them; never silently commit them.
- Note what is already staged — the user may have staged something on purpose; mention it.

### Step 2 — Discover commit conventions

Search, in order, and use the first source(s) that define rules (co-authorship is never taken from these
sources — see Hard Rule 5):

1. Explicit user instructions in the current conversation (e.g. "don't use co-authorship").
2. Repo guidelines: `AGENTS.md`, `CLAUDE.md`, `.junie/AGENTS.md`, `.junie/guidelines.md`, `CONTRIBUTING.md`,
   `docs/CONTRIBUTING*`, `README*` sections about commits, `.github/` (PR templates, workflows).
3. Tooling config: `commitlint.config.*`, `.commitlintrc*`, `.czrc`, `.cz.toml`, `package.json`
   (`commitlint` / `config.commitizen`), `lefthook.yml`, `.pre-commit-config.yaml`, `.husky/commit-msg`,
   `.gitmessage` / `git config commit.template`.
4. De-facto practice from `git log` (last ~30 commits): prefix style (`feat(scope):`, `[area]`, ticket IDs
   like `KT-12345`), allowed types (e.g. `misc:`), scopes in use, capitalization after the colon,
   imperative mood, subject length, body usage, trailers (ignore `Co-authored-by` ones).

**Fallback** (nothing found): Conventional Commits, one line, no body, no trailers:
`<type>(<optional scope>): <Imperative summary>` — types `feat`, `fix`, `refactor`, `perf`, `test`,
`docs`, `build`, `ci`, `chore`, `style`, `revert`; subject ≤ 72 chars, no trailing period.

State briefly which convention you detected and where it came from. If the repo uses or requires
co-authorship trailers, mention that they will still be omitted per Hard Rule 5.

### Step 3 — Group changes into commits

Best practices:

- **One logical change per commit**; each commit should be understandable and revertable on its own.
- **Each commit should build** (and ideally pass tests). Dependency/lockfile changes go with the code
  that needs them; renames go with the refactor that motivates them.
- Order commits so dependencies come first: refactors/infrastructure → features/fixes → docs.
- Keep docs in a separate commit unless conventions say docs travel with code.
- A single file may be split across commits at hunk/line level when it contains unrelated changes.
- Do not over-split: trivial related edits belong together.

### Step 4 — Propose and wait for approval

Present the plan and **stop**. Format:

```
Detected convention: <convention> (source: <file / git log / fallback>)

1. `<commit message>`
   - <path> — <what changes, one line>
   - <path> (partial: <which hunks>) — …
2. `<commit message>`
   - …

Excluded (left uncommitted): <paths>   ← or "none"
Notes: <line-level splits, already-staged files, risks, anything needing a decision>

Approve to commit, or tell me what to change.
```

Then end the turn and wait for the user's reply (use the user-question tool if available, otherwise end
with the question). On each reply:

- Change requests → update the plan, re-present the **full** updated list, ask again.
- Questions or concerns → answer them, ask again. Do not treat "ok, but…" as approval.
- Explicit, unconditional approval → proceed to Step 5 with exactly the approved plan.

### Step 5 — Commit

1. Start from a clean index: `git reset -q` (only unstages; working tree untouched). Unstage excluded
   files if needed.
2. For each commit in order:
   - Stage whole files: `git add -- <paths>` (use `git add -A -- <dir>` for renames/deletions).
   - Stage partial files via "Line-level splits" below.
   - Check `git diff --cached --stat -M` matches the plan.
   - Commit with the approved message: `git commit -m "<subject>"` (add `-m "<body>"` only if the
     convention uses bodies; add trailers only if the convention requires them — never
     `Co-authored-by`, see Hard Rule 5).
   - After committing, check `git log -1 --format=%B` contains no `Co-authored-by` line (commit
     templates or hooks may inject one); if it does, report it to the user instead of rewriting history.
3. Optional but recommended for multi-commit series touching code: verify an intermediate commit builds in
   a temporary worktree (`git worktree add --detach <tmp> <sha>`, build there, then
   `git worktree remove --force <tmp>`), reusing the main build cache where possible.

#### Line-level splits

Never edit the working file. Build the intermediate version in a temp file and write it straight into
the index:

```bash
# produce $TMP/file = HEAD version + only the hunks for this commit (script, or patch apply)
git show HEAD:path/to/file > "$TMP/file"     # or construct from the working copy
# ... apply the selected hunks to "$TMP/file" ...
h=$(git hash-object -w "$TMP/file")
git update-index --cacheinfo 100644,"$h",path/to/file
```

Alternatively: `git diff -- path > $TMP/p.diff`, keep only the wanted hunks, then
`git apply --cached $TMP/p.diff`. The final commit then stages the full working file normally.

### Step 6 — Report

```bash
git log --stat -M --format='--- %h %s' -<N>
git status --short
```

Report: list of created commits (`<short sha> <subject>`), what was left uncommitted, any line-level
splits, verification performed (and what wasn't, e.g. "intermediate commit not built separately"), and
any leftover processes or temp worktrees.
