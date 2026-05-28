# ultra-review-local

A local, multi-agent code-review skill for [Claude Code](https://claude.com/claude-code):
a free, fully-local equivalent of the cloud `/ultrareview` feature. Six specialized
reviewers run in parallel against your branch's changes (or the whole project if there
are none), then a verifier dedupes and confirms each finding before producing a single
prioritized markdown report.

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

## Differences from cloud `/ultrareview`

- **Runs locally** — no cloud sandbox, no billed compute beyond your own Claude Code
  subscription.
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
