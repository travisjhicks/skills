# claude-isms

An [Agent Skill](https://agentskills.io) that removes the recognizable
house dialect of AI-assistant prose ("footgun," "blast radius," "You're absolutely
right," praise tics, em dash overuse) from everything Claude writes for a human
reader: answers, plans, reports, review findings, commit messages, PR bodies,
documentation, and issue-tracker comments.

## Installing

Install this skill with the [`skills` CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add travisjhicks/skills --skill claude-isms
```

Add `-g` to install it for your user account instead of the current project, and
`-a claude-code` or `-a codex` to skip the agent prompt. See the
[repository README](../../README.md#installing) for other options.

## What it does

The skill is a word-choice and voice guide, applied on every response without being
asked. It catalogs roughly 250 terms grouped by how grating they are, on a 1-to-5
annoyance scale: 5 and 4 are bans, 3 and 2 are strong defaults, 1 is "vary rather
than lean on." Each entry gives plain replacements; often the right edit is to
delete the word entirely.

More useful than the list itself is the taxonomy behind it. The entries are
instances of a small set of recurring habits, each named and tagged: grading the
user instead of answering (praise tics), narrating the process (speech tics),
casting code as a war zone (danger metaphors) or a crime scene (detective
register), giving tools feelings, compressing words into opaque shortenings, and
importing consulting language, among others. When a word fits one of these families
but is not listed, the same rule applies: say the plain thing. The tie-breaker is
the editor test: would a careful human editor who has never read an engineering
blog keep this word?

The rules govern prose only. Identifiers, API names, file paths, and quoted output
stay as written, and when a listed word is the exact technical name of the thing
under discussion, the name stays.

## Usage

Nothing to invoke. The skill's description tells Claude to apply it on every
response and every follow-up turn, including short or tool-heavy ones. Explicit
instructions from the user or the project always win over the ruleset.

## How it's built

`SKILL.md` is self-contained: the scope rules, the habit taxonomy, and the full
graded term list with replacements. See [`SKILL.md`](SKILL.md) for the complete
list.
