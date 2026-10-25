# intent-reconstruct

An [Agent Skill](https://agentskills.io) that performs a **blind,
clean-room reconstruction** of any document — a prompt, code, procedural
instructions, poetry, documentation, or configuration. It distills the document
into an abstract intent specification, has an isolated agent that never sees the
original rebuild the document from that spec alone, then verifies how much of the
intent survived.

## Installing

Install this skill with the [`skills` CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add travisjhicks/skills --skill intent-reconstruct
```

Add `-g` to install it for your user account instead of the current project, and
`-a claude-code` or `-a codex` to skip the agent prompt. See the
[repository README](../../README.md#installing) for other options.

## What it does

Three isolated roles run in sequence:

1. **Extractor** — reads the original and writes `<stem>.intent.md`: an abstract
   specification addressed to an author who will never see the original. It
   re-describes every behavior in its own words, calls out the handful of
   *intent-bearing tokens* that must survive verbatim (glyphs, format strings,
   trigger phrases, API names), flags intentional quirks to preserve, and states
   type-appropriate fidelity criteria (behavioral rules for a prompt; contracts
   and algorithms for code; meaning, form, and meter for a poem).
2. **Blind generator** — receives only the spec's content, inlined into its
   prompt. It never learns the original's filename, path, or repository (the spec
   is written to contain none of them), writes its output outside the source
   tree, and performs no file reads. Blindness is structural, not a promise.
3. **Verifier** — reads everything, scores intent coverage against the spec's own
   criteria, and audits prose overlap to confirm the reconstruction wasn't
   quietly copied. On failure, a bounded refinement loop (three generation passes
   at most) repairs the spec or re-directs the generator — using only abstract
   gap descriptions, so blindness is never broken.

Outputs land beside the source: `<stem>.intent.md`, `<stem>.regenerated.<ext>`,
and `<stem>.fidelity-report.md`. The original is never modified.

## Why blindness matters

If a blind agent can rebuild the document from the spec alone, the spec is
*proven sufficient* — the document's intent has been fully externalized. That
makes the spec independently useful (documentation, de-identification, porting),
and makes the reconstruction genuinely independent wording rather than a
paraphrase. Personal data is replaced with declared placeholders by default, so
the rebuilt document is de-identified and shareable.

## Usage

```text
/intent-reconstruct prompts/my-system-prompt.txt
Blindly recreate this file from its intent: scripts/backup.sh
Write an intent spec for this poem and rebuild it clean-room.
```

## How it's built

`SKILL.md` is the entry point: validation, the three verbatim role prompts, the
refinement loop, and the edge-case table. `references/intent-spec-guide.md`
teaches the Extractor what a sufficient spec contains per document type;
`references/fidelity-report-template.md` gives the Verifier its report structure
and the gap-ledger rules that keep refinement blind.
