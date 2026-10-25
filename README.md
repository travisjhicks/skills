# Skills

A personal collection of [Agent Skills](https://docs.claude.com/en/docs/claude-code/skills)
for [Claude Code](https://claude.com/claude-code).

Each skill lives in its own directory under [`skills/`](skills/), with a `SKILL.md`
(the agent-facing entry point Claude loads) and a `README.md` (a human-facing overview
for browsing here). Click a skill below for its full README.

## Skills

| Skill | What it does |
| --- | --- |
| [**claude-isms**](skills/claude-isms/README.md) | Word-choice and voice rules that remove the house dialect of AI-assistant prose ("footgun," "blast radius," "You're absolutely right," praise tics, em dash overuse) from everything Claude writes for a human reader. |
| [**freddish**](skills/freddish/README.md) | Translates prose into "Freddish," the preschooler-friendly language from *Mister Rogers' Neighborhood*, by applying its nine rules in a single careful pass. |
| [**intent-reconstruct**](skills/intent-reconstruct/README.md) | Blind clean-room reconstruction of any document: distills it into an intent specification, has an isolated agent that never sees the original rebuild it from the spec alone, then verifies intent coverage and audits for leakage. |
| [**simplified-technical-english**](skills/simplified-technical-english/README.md) | Rewrites technical prose into a controlled English for software—based on the concepts of ASD-STE100 but its own style: one word per meaning, one instruction per sentence, active voice, capped sentence lengths, with registers and protections built for software text. |
| [**ultra-review-local**](skills/ultra-review-local/README.md) _(deprecated)_ | Local, multi-agent code review (six specialized reviewers + a verifier). **Superseded by the consolidated `/code-review max`** (optionally with `/effort ultracode`); kept for reference. |

## Installing

The skills install with the [`skills` CLI](https://github.com/vercel-labs/skills),
which needs Node.js 22.20 or later. It asks which agents to install to (Claude Code,
Codex, and others it detects) and installs into the current project unless you add
`-g` to install for your user account.

Install, choosing the skills, agents, and scope from prompts (the skill list has a
select-all option):

```bash
npx skills add travisjhicks/skills
```

List the skills in this repository:

```bash
npx skills add travisjhicks/skills --list
```

Install one skill, here for your user account:

```bash
npx skills add travisjhicks/skills --skill freddish -g
```

The deprecated `ultra-review-local` is marked internal, so the CLI hides it unless
`INSTALL_INTERNAL_SKILLS=1` is set.

### Installing by hand

Each skill is a self-contained directory, so you can also copy or symlink one into
your agent's skills folder:

| Agent | For your user account | For one project |
| --- | --- | --- |
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Codex | `~/.agents/skills/` | `.agents/skills/` |

```bash
ln -s "$PWD/skills/freddish" ~/.claude/skills/freddish
```

The agent loads the skill from its `SKILL.md`; the `README.md` and `references/`
travel with it.
