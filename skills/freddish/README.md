# freddish

A [Claude Code](https://claude.com/claude-code) skill that translates ordinary prose
into **"Freddish"**—the meticulously careful, preschooler-friendly language Fred
Rogers and his team developed for *Mister Rogers' Neighborhood*.

## What it does

Freddish translation applies **nine rules**: the first states the idea clearly in
words a preschooler knows, and the other eight each target one specific way adult
prose fails young children—negative framing, abstraction, directive tone, false
certainty, exclusionary assumptions, missing motivation, hidden value judgments, and
developmental disconnection. The skill applies all nine together in a single careful
pass, and also folds in Freddish's broader lens—catching words a literal-minded child
might misread (the classic example: a blood-pressure cuff that "blows up").

> **A note on architecture.** An earlier version dispatched each rule to its own
> isolated subagent, one rule at a time. We tested that against a single subagent given
> all nine rules at once, and the subagent-per-rule approach did **not** perform as
> well—isolated passes drifted off the input's meaning, and a step-by-step chain bled
> the worked example's wording into the output. The single all-rules-at-once pass
> produced the most coherent, on-topic translations, so that is what the skill does.

## Usage

Ask for a Freddish translation, a Mister Rogers-style rewrite, or simply describe the
goal:

```text
Translate this into Freddish: "Don't touch the stove, it's dangerous."
Rewrite this so a four-year-old won't be scared by it: ...
```

The skill returns the final Freddish translation (and can show the original alongside
it for reference).

## How it's built

`SKILL.md` is the entry point and authoritative spec: it determines the input,
substitutes it into the translation prompt, and dispatches the subagent. The
prompt—the nine rules with before/after examples plus the surrounding instructions—is
inline in `SKILL.md`; there are no other files to load.

See [`SKILL.md`](SKILL.md) for the full procedure, including why each rule exists.
