# freddish

A [Claude Code](https://claude.com/claude-code) skill that translates ordinary prose
into **"Freddish"** — the meticulously careful, preschooler-friendly language Fred
Rogers and his team developed for *Mister Rogers' Neighborhood*.

## What it does

Freddish translation is a deliberate, **nine-step rewriting process**. Each step
targets one specific way adult prose fails young children: negative framing,
abstraction, directive tone, false certainty, exclusionary assumptions, missing
motivation, hidden value judgments, and developmental disconnection.

The skill enforces **one rule at a time**: rather than asking a single pass to apply
all nine rules at once (which dilutes them), it dispatches each step to its own
subagent that sees only that step's instruction and the current text. A
literal-misinterpretation pre-pass runs first, flagging words a literal-minded child
might misread (the classic example: a blood-pressure cuff that "blows up") so you can
revise before the chain runs.

## Usage

Ask for a Freddish translation, a Mister Rogers-style rewrite, or simply describe the
goal:

```text
Translate this into Freddish: "Don't touch the stove, it's dangerous."
Rewrite this so a four-year-old won't be scared by it: ...
```

By default the skill shows the full progression (the text after each of the nine
steps) plus the final translation, so you can see how each rule peels away a different
adult-prose failure mode. Ask for "just the final translation" to skip the
intermediate steps.

## How it's built

`SKILL.md` is the orchestration entry point and authoritative spec. The prompts and
canonical step definitions live under `references/`:

```
references/
├── freddish.txt              # the nine steps with before/after examples
├── pre_pass_prompt.md        # literal-misinterpretation screening prompt
└── step_prompt_template.md   # per-step subagent prompt template
```

See [`SKILL.md`](SKILL.md) for the full procedure, including why each rule exists.
