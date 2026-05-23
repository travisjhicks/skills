# Reviewer output schema

Canonical schema for the YAML block each reviewer emits. The verifier parses
output against this; reviewer prompts reference this file rather than
re-spelling the schema, so any field change is made here once.

## Required shape

```yaml
reviewer: <one of: security | correctness | architecture | performance | tests | style>
findings:
  - severity: <critical | important | suggestion>
    file: <path/relative/to/repo/root>
    line: <integer — the cited line in the file; OMIT when the finding has no specific line>
    title: <short noun phrase, ~10 words>
    rationale: |
      One paragraph. Cite the surrounding code or behavior precisely so the
      verifier can confirm by reading the file. If you quote the exact line
      or snippet, the verifier can skip re-reading for `suggestion`-severity
      findings.
    suggested_fix: <one-line concrete fix; OMIT if you genuinely have no concrete suggestion>
coverage:
  files_inspected: <integer>
  notes: |
    Free-form. Mention skipped files, context-budget exhaustion, or any
    confidence caveats. Empty string is acceptable when nothing notable.
```

## Field rules

- **`severity`** — choose one of the three literal values. Don't invent
  intermediate severities.
- **`file`** — repo-root-relative path; no leading `./`. Use forward slashes
  even on Windows.
- **`line`** — omit the key entirely when the finding has no specific line
  (e.g. "no test exists for this behavior anywhere in the suite"). Do NOT
  emit `line: 0` or `line: null` — the verifier renders the file:line link
  without a line suffix when `line` is absent.
- **`title`** — fits on one line in a terminal. No trailing punctuation.
- **`rationale`** — one paragraph. If you can quote the offending line or
  the specific snippet you're flagging, do — that lets the verifier skip a
  re-read on low-severity findings.
- **`suggested_fix`** — optional but strongly preferred. One line. Omit
  rather than write a vague "consider refactoring".
- **`findings`** — empty list (`findings: []`) when you have nothing to
  report. Don't invent findings to look thorough.
- **`coverage.notes`** — record any context-budget exhaustion, files you
  skipped, or category-specific gaps the verifier should know about.

## Output rules

- Return **only** a fenced YAML block. No prose before or after — the verifier
  parses programmatically.
- The block must start with `reviewer:` and parse as valid YAML.
- If you produce zero findings, still emit the block with `findings: []` and
  a meaningful `coverage` entry.
