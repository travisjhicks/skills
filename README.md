# skills

A personal collection of [Agent Skills](https://docs.claude.com/en/docs/claude-code/skills)
for [Claude Code](https://claude.com/claude-code).

Each skill lives in its own directory under [`skills/`](skills/), with a `SKILL.md`
(the agent-facing entry point Claude loads) and a `README.md` (a human-facing overview
for browsing here). Click a skill below for its full README.

## Skills

| Skill | What it does |
| --- | --- |
| [**ultra-review-local**](skills/ultra-review-local/README.md) _(deprecated)_ | Local, multi-agent code review (six specialized reviewers + a verifier). **Superseded by the consolidated `/code-review max`** (optionally with `/effort ultracode`); kept for reference. |
| [**freddish**](skills/freddish/README.md) | Translates prose into "Freddish," the preschooler-friendly language from *Mister Rogers' Neighborhood*, by applying its nine rules in a single careful pass. |

## Installing a skill

Skills are self-contained directories. To make one available to Claude Code, symlink
(or copy) its directory into your skills folder:

```bash
ln -s "$PWD/skills/freddish" ~/.claude/skills/freddish
```

Claude loads the skill from its `SKILL.md`; the `README.md` and `references/` travel
with it.

## License

MIT. Each skill bundles its own license—see the `LICENSE.txt` in
[`skills/ultra-review-local/`](skills/ultra-review-local/LICENSE.txt) and
[`skills/freddish/`](skills/freddish/LICENSE.txt).
