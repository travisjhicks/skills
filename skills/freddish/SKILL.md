---
name: freddish
description: Translates user-supplied prose into "Freddish," the meticulously careful preschooler-friendly language Fred Rogers' team developed for Mister Rogers' Neighborhood, by chaining nine independent subagent passes. Use this skill whenever the user asks for a Freddish translation, a Mister Rogers-style rewrite for young children, or asks to apply the Freddish/nine-step rules to a sentence or paragraph, even if they don't name the process explicitly but describe wanting to rephrase something so a preschooler would understand it.
---

# Freddish Translator

"Freddish" is the in-house name for the language Fred Rogers and his collaborators developed for *Mister Rogers' Neighborhood*. Translating into Freddish is a deliberate, nine-step rewriting process. Each step targets one specific way that ordinary adult prose fails preschoolers: negative framing, abstraction, directive tone, false certainty, exclusionary assumptions, missing motivation, hidden value judgments, and developmental disconnection.

## When this skill applies

Use this skill when the user gives you a sentence or short paragraph and asks for a Freddish translation, a Mister Rogers-style rewrite, or otherwise asks for the Freddish rules to be applied. Also use it when the user describes the underlying goal (e.g. "rephrase this so a four-year-old would understand and not be scared") without naming Freddish. The nine-step process is what they want, even if they don't know its name.

If it's unclear whether the user wants the full nine-step process or just a single kid-friendly rewrite, ask briefly before proceeding.

## What Freddish is fundamentally doing

The nine steps serve a deeper goal: anticipating where a preschooler's mind might go when they hear adult prose. The formal targets—positive framing, softened certainty, inclusive language, added motivation, developmental anchoring—are what the rules systematically catch. But the underlying lens is broader: it also includes catching words and phrases a literal-minded child might misread. Fred Rogers's team famously had a nurse change "I'm going to blow this up" (a blood-pressure cuff) to "I'm going to puff this up with some air," because "blow it up" might sound like an explosion. A pet being "put to sleep" was excised for fear children would become afraid of falling asleep themselves. "Down the drain" got a whole song ("You Can Never Go Down the Drain") because drains might otherwise seem to exist to suck things down.

Keep this lens active when reasoning about edge cases. The nine rules are the structured part of Freddish; the underlying question is always, "How might a four-year-old actually hear this?" The pre-pass below catches part of what the nine rules don't; the rest is judgment.

## Core principle: one rule at a time

The nine rules must not be applied simultaneously. Each rule is a focused pass that addresses one specific failure mode, and combining rules dilutes them: a single LLM pass asked to "apply all nine rules at once" will inevitably skip, conflate, or shortcut steps. The translation only works when each rule has the input text's full attention.

To enforce this separation, dispatch each step to its own subagent. The subagent sees only that step's instruction and the current text. It cannot anticipate later steps because it doesn't know they exist for that call.

## The procedure

1. Read `reference/freddish.txt` to load the nine steps with their before/after examples.
2. Run the **literal-misinterpretation pre-pass** subagent (see "Spawning the pre-pass" below) on the user's original prose. If it returns flagged items, present them to the user and ask whether to revise the input first or proceed as-is. Wait for the answer before continuing. If it returns "No hazards flagged.", continue without interruption.
3. Treat the user's input (revised or original) as the "current text."
4. Run nine **sequential** subagent calls, one per step, in order from 1 to 9. The output of step N becomes the input to step N+1. Do not run these in parallel; each step depends on the previous step's output.
5. After step 9 returns, present the final Freddish translation to the user, along with the intermediate results so the progression is visible.

### Spawning the pre-pass

The pre-pass screens the input for words or phrases a literal-minded preschooler might misread in a scary or confusing way: the class of hazard the formal nine-step rules don't directly catch. It does **not** transform the text; it surfaces hazards so the user can decide whether to revise before the chain runs.

Use the Agent tool with `subagent_type: "general-purpose"` and no `model` parameter (same model-inheritance reasoning as the nine steps). Prompt:

```
You are doing a Freddish pre-screen: checking text for words or phrases
that a preschooler might interpret literally in a way that could confuse
or scare them. This is separate from the nine-step Freddish rewriting
process; your job is to flag potential hazards for human review, not to
rewrite anything.

Fred Rogers's team famously caught issues like:
- "I'm going to blow this up" (a blood-pressure cuff) → might evoke an explosion
- A pet "put to sleep" → might make falling asleep frightening
- "Down the drain" → drains might seem to exist to suck things down
- A doctor "looking inside your eyes" → might suggest seeing thoughts

Look for: idioms a literal-minded child might misread; words with scary
alternate meanings ("blow up", "kick", "die", "break", "tied up",
"freezing", "starving"); euphemisms (especially around death, illness,
or going away); and any phrasing that could conjure a frightening image
when taken at face value.

Text to screen:
<the user's original prose>

If you find hazards, return a short list. For each, give:
- The exact phrase from the text
- The literal misinterpretation a preschooler might make
- A one-line suggested alternative

If you find nothing, return exactly: No hazards flagged.

Return only the list (or the "No hazards flagged." line). No preamble,
no commentary, no markdown headers.
```

If the pre-pass returns "No hazards flagged.", proceed straight to step 1. If it returns flagged items, present them to the user and wait for the answer before starting the chain. `AskUserQuestion` is a good fit, offering "Revise input first" vs. "Proceed as-is."

### Spawning each step

Use the Agent tool with `subagent_type: "general-purpose"`. Do **not** pass the `model` parameter; leaving it unset causes the subagent to inherit the parent agent's model, which is what you want here (the user expects the step subagents to run at the same model tier as the parent).

The prompt for each subagent should follow this shape:

```
You are performing Step <N> of the Freddish translation process, the
preschooler-friendly rewriting method developed for Mister Rogers' Neighborhood.

Your single job is Step <N>. Do not apply any other step. Do not anticipate
later steps. Do not combine concerns. Make exactly the transformation this
step describes, and nothing else.

Step <N> instruction:
<paste the step's full instruction line from reference/freddish.txt>

Example of this step applied in isolation:
Before: "<paste the Before line from freddish.txt>"
After:  "<paste the After line from freddish.txt>"

Current text to transform:
<the current working text>

Return the full rewritten text and only the rewritten text. No preamble, no
commentary, no quotation marks, no diff, no explanation of what you changed.
Just the new text on its own.
```

Reading the step instruction and example freshly from `reference/freddish.txt` for each call (rather than memorizing them) keeps the prompts grounded and lets the reference file be the single source of truth.

### Presenting the result

After all nine subagents have returned, show the user the progression and then the final result, formatted like:

```
Original: <user's input>
Step 1:   <text after step 1>
Step 2:   <text after step 2>
Step 3:   <text after step 3>
Step 4:   <text after step 4>
Step 5:   <text after step 5>
Step 6:   <text after step 6>
Step 7:   <text after step 7>
Step 8:   <text after step 8>
Step 9:   <text after step 9>

Freddish translation: <final text from step 9>
```

The intermediate steps are valuable: they show how each rule peels away a different adult-prose failure mode. Include them by default. If the user explicitly asks for "just the final translation," skip the intermediate lines.

## Why each rule exists

Briefly, so you can reason about edge cases the subagents might handle awkwardly:

- **Step 1**: clarity in preschooler-accessible vocabulary. The starting point.
- **Step 2**: positive framing. Negative statements ("don't…", "it is dangerous to…") prime fear; positive ones invite the child toward safety.
- **Step 3**: redirect to trusted authorities. Preschoolers can't yet weigh abstract principles; they trust people.
- **Step 4**: remove prescriptive/directive language. Commands feel coercive and can produce resistance.
- **Step 5**: soften certainty. "Will" over-promises; "can" leaves room for the child's reality.
- **Step 6**: universalize. Not all children have parents, two parents, the parents they live with, etc. "Favorite grown-ups" reaches every child.
- **Step 7**: add motivation. A rule without a reason rarely sticks.
- **Step 8**: strip value judgments. "Good" carries adult moral weight; "important" is more neutral and developmentally honest.
- **Step 9**: anchor to development. Connecting the message to growing up gives a preschooler a place to file it.

If a step's output looks like it didn't change much from the input, that's usually fine; not every step needs to alter the text. Pass the unchanged text forward to the next step rather than re-running.
