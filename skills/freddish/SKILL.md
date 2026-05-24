---
name: freddish
description: Translates user-supplied prose into "Freddish," the meticulously careful preschooler-friendly language Fred Rogers's team developed for Mister Rogers' Neighborhood, by chaining nine independent subagent passes. Use this skill whenever the user asks for a Freddish translation, a Mister Rogers-style rewrite for young children, or asks to apply the Freddish/nine-step rules to a sentence or paragraph, even if they don't name the process explicitly but describe wanting to rephrase something so a preschooler would understand it.
license: Complete terms in LICENSE.txt
argument-hint: [text to translate]
---

# Freddish Translator

"Freddish" is the in-house name for the language Fred Rogers and his collaborators developed for *Mister Rogers' Neighborhood*. Translating into Freddish is a deliberate, nine-step rewriting process. Each step targets one specific way that ordinary adult prose fails preschoolers: negative framing, abstraction, directive tone, false certainty, exclusionary assumptions, missing motivation, hidden value judgments, and developmental disconnection.

## When this skill applies

Use this skill when the user gives you a sentence or short paragraph and asks for a Freddish translation, a Mister Rogers-style rewrite, or otherwise asks for the Freddish rules to be applied. Also use it when the user describes the underlying goal (e.g. "rephrase this so a four-year-old would understand and not be scared") without naming Freddish. The nine-step process is what they want, even if they don't know its name.

If it's unclear whether the user wants the full nine-step process or just a single kid-friendly rewrite, ask briefly before proceeding.

## Input shape

Freddish is calibrated for sentences and short paragraphs — the canonical example in `references/freddish.txt` is a single sentence. For longer or more complex input (multi-paragraph passages, dialogue, lists), still run the full chain. Do not refuse the input, and do not ask the user to chunk it for you: if they explicitly asked for the skill to run on a larger block of text, the right answer is to process it as best as the chain can while staying true to the intent of the translation. Preserve paragraph and sentence boundaries as you transform.

A few notes for non-trivial input:

- The orchestrator passes the entire current text to each step subagent, so long input multiplies the per-step prompt size. Mention the dispatch cost to the user if the input is unusually long (e.g. several thousand words) so they aren't surprised by latency or token usage, but proceed unless they tell you to stop.
- Step 7 ("add a simple motivational idea") is the step most sensitive to input length. For a multi-sentence passage, the subagent should add a single motivation that applies to the passage as a whole rather than one motivation per sentence — otherwise the output bloats with parallel reasons.
- Steps that do not apply to the current text (e.g. Step 8's value-judgment removal on text with no value judgments) should return the text unchanged. The per-step prompt template in `references/step_prompt_template.md` already tells the subagent this is acceptable.

## What Freddish is fundamentally doing

The nine steps serve a deeper goal: anticipating where a preschooler's mind might go when they hear adult prose. The formal targets—positive framing, softened certainty, inclusive language, added motivation, developmental anchoring—are what the rules systematically catch. But the underlying lens is broader: it also includes catching words and phrases a literal-minded child might misread. Fred Rogers's team famously had a nurse change "I'm going to blow this up" (a blood-pressure cuff) to "I'm going to puff this up with some air," because "blow it up" might sound like an explosion. A pet being "put to sleep" was excised for fear children would become afraid of falling asleep themselves. "Down the drain" got a whole song ("You Can Never Go Down the Drain") because drains might otherwise seem to exist to suck things down.

Keep this lens active when reasoning about edge cases. The nine rules are the structured part of Freddish; the underlying question is always, "How might a four-year-old actually hear this?" The pre-pass below catches part of what the nine rules don't; the rest is judgment.

## Core principle: one rule at a time

The nine rules must not be applied simultaneously. Each rule is a focused pass that addresses one specific failure mode, and combining rules dilutes them: a single LLM pass asked to "apply all nine rules at once" will inevitably skip, conflate, or shortcut steps. The translation only works when each rule has the input text's full attention.

To enforce this separation, dispatch each step to its own subagent. The subagent sees only that step's instruction and the current text. It cannot anticipate later steps because it doesn't know they exist for that call.

## The procedure

1. **Determine the input text.** If prose was provided with the invocation (text after `/freddish`, or anywhere in the user's message), use it as the original prose. If no text was provided, ask the user what they'd like translated into Freddish—and, if it's unclear, whether they want the full nine-step chain or a single rewrite—then wait for their reply. Do not read `references/freddish.txt`, run the pre-pass, or run any step until you have text to translate.
2. Read `references/freddish.txt` once to load the nine steps with their before/after examples. Cache the content for the rest of the procedure — each step's prompt only needs that step's slice, but the orchestrator extracts each slice from this single cached read rather than re-reading the file per step.
3. Run the **literal-misinterpretation pre-pass** subagent (see "Spawning the pre-pass" below) on the user's original prose. The pre-pass section below describes how to normalize the response and when to interrupt the user.
4. Treat the user's input (revised or original) as the "current text."
5. Run nine **sequential** subagent calls, one per step, in order from 1 to 9. The output of step N becomes the input to step N+1. Do not run these in parallel; each step depends on the previous step's output.
6. After step 9 returns, present the final Freddish translation to the user, along with the intermediate results so the progression is visible.

### Spawning the pre-pass

The pre-pass screens the input for words or phrases a literal-minded preschooler might misread in a scary or confusing way: the class of hazard the formal nine-step rules don't directly catch. It does **not** transform the text; it surfaces hazards so the user can decide whether to revise before the chain runs.

Read `references/pre_pass_prompt.md` for the prompt body. Substitute the user's original prose between the `<<<USER_TEXT_BEGIN>>>` / `<<<USER_TEXT_END>>>` sentinels (the sentinels mark the prose as data, not instructions — see "On subagent_type and model" below). Send the prompt via the Agent tool with `subagent_type: "general-purpose"` and no `model` parameter.

Normalize the response before deciding how to proceed: strip surrounding whitespace, lowercase, and drop trailing punctuation. If the normalized response equals `no hazards flagged` or is empty, proceed straight to step 1 without interruption. Otherwise treat the response as a flagged-items list, present it to the user, and wait for the answer before continuing. `AskUserQuestion` is a good fit, offering "Revise input first" vs. "Proceed as-is."

If the user selects "Revise input first," collect their revised prose and re-run the pre-pass once on the revision. If that second pass also flags items, surface them but proceed to step 1 regardless — do not loop a third time, to bound cost and avoid soft deadlocks.

### Spawning each step

Read `references/step_prompt_template.md` for the prompt body and the placeholder substitution rules. For each step N from 1 to 9, fill in:

- `{{N}}` — the step number
- `{{step_instruction}}` — the `Step N. …` instruction sentence(s) from your cached read of `references/freddish.txt`. Do not include the `Before:` / `After:` lines — those go in the next two placeholders.
- `{{before_example}}` and `{{after_example}}` — the text from the matching `Before:` and `After:` lines.
- `{{current_text}}` — the user's prose for step 1, or the previous step's output for steps 2-9.

Send the filled prompt via the Agent tool with `subagent_type: "general-purpose"` and no `model` parameter (see "On subagent_type and model" below). Pass the result forward as the input to step N+1.

### On subagent_type and model

The skill uses `subagent_type: "general-purpose"` because no narrower subagent type is currently available for pure text-rewriting work, and the Agent tool has no parameter for restricting tool access at spawn time. The prompt-injection mitigation is the sentinel delimiting documented in `references/pre_pass_prompt.md` and `references/step_prompt_template.md`: the subagent is told to treat sentinel-wrapped content as data rather than instructions, even when that content appears instruction-like. If a future Claude Code release adds a narrower subagent type or per-spawn tool restriction, switch to that.

The `model` parameter is intentionally omitted so each subagent inherits the parent orchestrator's model — the user expects the chain to run at the same tier as the rest of their session. Caveat: if the parent is on a small/fast tier (e.g. Haiku-class), nine sequential quality-sensitive rewrites compound at that tier and the output may be noticeably less faithful to the Freddish style. Mention this to the user up front if you notice the parent model is small; also note that the procedure dispatches roughly ten subagent calls per invocation (one pre-pass + nine steps), which is not free.

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
