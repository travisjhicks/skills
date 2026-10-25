# ultra-review-local

> **⚠️ Deprecated — use `/code-review max` instead.** This skill predates the
> consolidated `/code-review` command, which now provides a more powerful local
> multi-agent review: **`/code-review max`** (optionally with `/effort ultracode`).
> Prefer that for new reviews. The skill is kept here for reference and still runs
> if you invoke `/ultra-review-local` explicitly.

A multi-agent code-review skill for [Claude Code](https://claude.com/claude-code):
Claude Code orchestrates the review from your local session—spawning the reviewer and
verifier subagents itself—instead of handing the job to a cloud sandbox the way the
cloud `/code-review ultra` (formerly `/ultrareview`) does. Six specialized reviewers
run in parallel against your branch's changes (or the whole project if there are none),
then a verifier dedupes and confirms each finding before producing a single prioritized
markdown report.

## Installing

Install this skill with the [`skills` CLI](https://github.com/vercel-labs/skills):

```bash
INSTALL_INTERNAL_SKILLS=1 npx skills add travisjhicks/skills --skill ultra-review-local
```

Add `-g` to install it for your user account instead of the current project, and
`-a claude-code` or `-a codex` to skip the agent prompt. See the
[repository README](../../README.md#installing) for other options.

This skill is marked internal, so the CLI lists and installs it only when
`INSTALL_INTERNAL_SKILLS=1` is set.

## What it does

When invoked via `/ultra-review-local`, the skill:

1. **Detects scope automatically** — staged + unstaged changes, branch ahead of `main`,
   or the whole project—and announces it before spending budget.
2. **Spawns six reviewer agents in parallel**, each specialized:
   - **security** — injection, authz, secrets, crypto, unsafe defaults
   - **correctness** — logic bugs, edge cases, null/undefined, races, error paths
   - **architecture** — layering, dependency direction, premature abstractions
   - **performance** — N+1s, blocking I/O, unbounded reads, complexity
   - **tests** — missing coverage, weak assertions, flaky patterns
   - **style** — convention drift, naming, dead code
3. **Runs a verifier pass** that reads the actual code to confirm each finding,
   deduplicates across reviewers, tightens severity, and drops findings it can't
   reproduce—aiming for a low false-positive bar.
4. **Prints a single markdown report** grouped by Critical / Important / Suggestions,
   with `file:line` links.

Every agent runs on `model: "opus"`. This skill is meant to be thorough, not fast.

## Usage

```text
/ultra-review-local                # auto-detect scope
/ultra-review-local src/auth/      # restrict to a path
/ultra-review-local main..HEAD     # explicit revision range
/ultra-review-local whole          # force whole-project review
/ultra-review-local staged         # force staged-only review
/ultra-review-local --only security,tests
/ultra-review-local --skip style
/ultra-review-local --out review.md
```

## Scope detection

With no scope arg, the skill auto-detects, stopping at the first match:

1. **Unborn HEAD** — a brand-new repo with no commits yet → review the staged +
   untracked files.
2. **Uncommitted work** — anything in `git status --porcelain` → review `git diff HEAD`
   plus untracked files.
3. **Detached HEAD** — a branch comparison doesn't apply, so fall through to whole-project.
4. **Branch ahead of default** — commits not in `origin/HEAD` (falling back to `main`
   → `master`) → review the branch diff.
5. **Whole project** — clean repo with no diff (or no default branch resolved) → review
   the tracked tree (excludes `node_modules/`, `dist/`, `build/`, lockfiles, etc.).

You can always override with an explicit arg. `SKILL.md` is the authoritative spec
for the exact order and edge-case handling.

## Differences from cloud `/code-review ultra`

- **Orchestrated locally** — no separate cloud sandbox to provision; Claude Code runs
  the reviewers and verifier from your normal session, which avoids the added expense
  of a cloud-based review.
- **No GitHub remote required** — works on any local git repo, including ones with no
  remote at all.
- **Whole-project fallback** — if there's no diff vs. the default branch, it still
  produces a review.
- **Single verifier instead of per-finding verifiers** — simpler and cheaper, still
  cuts most false positives.
- **Visible & tweakable** — every reviewer and the verifier is a markdown prompt under
  `references/` that you can edit to retune what gets flagged.

## How it's built

`SKILL.md` is the orchestration entry point and the authoritative spec for how the
skill runs (argument parsing, scope detection, dispatch, prompt-injection handling).
The prompts live under `references/`:

```
references/
├── verifier.md          # post-fanout aggregator that dedupes and confirms
└── reviewers/
    ├── _schema.md       # canonical YAML schema every reviewer emits
    ├── security.md
    ├── correctness.md
    ├── architecture.md
    ├── performance.md
    ├── tests.md
    └── style.md
```

Tweak any of them to retune what gets flagged in your codebase. See
[`SKILL.md`](SKILL.md) for the full orchestration detail.
