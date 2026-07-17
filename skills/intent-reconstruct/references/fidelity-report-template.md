# Fidelity Report Structure

This is the structure the **Verifier** follows for its report. Fill every section; write "none" rather than omitting a section, so absence is distinguishable from oversight. The report is a rendered Markdown document — use real em dashes and normal prose.

The Verifier produces two things from one analysis: this human-facing report, and (on FAIL) the **gap ledger** returned to the orchestrator to drive the refinement loop. The constraint that makes refinement blindness-safe: gap-ledger entries are written as *additions to the intent spec*, in the spec's abstract register, and must never quote the original beyond already-licensed tokens.

## Template

```markdown
# Fidelity Report — Blind Reconstruction of `<original filename>`

**Date:** <YYYY-MM-DD>
**Method:** Intent-driven blind reconstruction (extract intent → blind generate → verify)
**Generation passes:** <N of 3>
**Final verdict:** ✅ PASS | ❌ FAIL

## Files

| File | Role |
|---|---|
| `<original>` | Original source document (left untouched) |
| `<stem>.intent.md` | Intent specification — the only bridge to the blind generator |
| `<stem>.regenerated.<ext>` | The blind reconstruction under review |

## Method

<Three-role summary: Extractor read the original and wrote the spec; the blind
generator received only the spec and never the original's path or name; the
Verifier read both and scored coverage. Note this is pure-intent reconstruction:
rosters, examples, and placeholder data are expected to diverge — the success
criterion is intent coverage, not byte equivalence.>

## Verdict by dimension

<One subsection per fidelity criterion declared in the spec's "Fidelity criteria
for this document type" section — the spec's own rubric, not one you invent.
For each: full / partial / missing, with specifics. End each subsection with
explicit `missing:` / `weakened:` lists (or "none").>

## Intent-bearing tokens and preserved quirks

<Checklist: every token and quirk the spec called out, each ✅/❌ with the
evidence.>

## Blindness / leakage audit

<Scan both documents for shared sentences and phrase runs of roughly 8+ words.
Classify each overlap as LICENSED (declared intent-bearing tokens, preserved
quirks, licensed verbatim blocks) or SUSPECTED LEAKAGE. Rate overall leakage
low / moderate / high with examples. If the spec declared licensed verbatim
blocks, note that the audit is limited in scope for those spans. For very short
documents, note that the audit is near-vacuous.>

## Expected, non-penalized divergences

<Differences the pure-intent rubric predicts and does not score as defects:
fresh examples, placeholder personal data, roster entries beyond the declared
distinctive ones, surface wording in creative works.>

## Gap ledger

<On PASS: "none". On FAIL, one entry per gap:>

| # | Gap (abstract addendum phrasing) | Classification | Severity |
|---|---|---|---|
| 1 | <Addendum-ready sentence, in the spec's own abstract register — never quoting unlicensed original prose> | spec omission \| generator miss | blocking \| minor |

<"Spec omission" = the spec never conveyed it (Extractor's gap; the addendum
must be appended to the spec). "Generator miss" = the spec conveyed it and the
reconstruction dropped or weakened it (cite the spec passage). If this is a
later pass, also record the prior passes' ledgers under "Pass history".>

## Minor cosmetic nits

<Optional polish that is not an intent failure — internal inconsistencies in a
fresh example, awkward phrasing. Never blocking.>

## Summary

<Two to four sentences: does the reconstruction do the same job? How independent
is the wording? What would a user need to change before using it (e.g. swap
placeholders for real data)?>
```

## Scoring guidance

- **PASS** requires: no `missing` entries on any dimension, every intent-bearing token and preserved quirk reproduced, and leakage rated low (or entirely licensed). `weakened` entries and cosmetic nits are judgment calls — fail on them only when the weakening would change behavior or meaning.
- Score against the spec's declared criteria, not against your own taste. If the spec itself failed to demand something the original clearly needs, that is still a gap — but classify it as a *spec omission* so the refinement loop repairs the spec rather than blaming the generator.
- Divergence is the default expectation, not a smell. The question is never "does it match?" but "does everything the spec demanded survive?"
