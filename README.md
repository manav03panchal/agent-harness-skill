# Manav's Agent Harness Skill

A skill for coding agents. It reworks a codebase, greenfield or legacy, so
agents can do reliable, high-throughput work in it. The repository becomes
the agent's environment: a map-style entry file, `docs/` as the system of
record, architecture that tooling enforces, an app and logs the agent can
read, and continuous garbage collection.

The approach comes from OpenAI's "Harness engineering" post. This skill
makes it harness-agnostic. It works with Claude Code, Codex, Cursor,
Copilot, Gemini CLI, or any tool that reads a repository-local instruction
file.

## Install

One command. It installs the skill into every agent on the machine (Claude
Code, Codex, Cursor, Copilot, Gemini CLI, and others it detects):

```bash
npx skills add manav03panchal/agent-harness-skill --all -g
```

Drop `-g` to install into the current project instead. The project then
carries the skill for the whole team.

Install the companion vetting skill the same way:

```bash
npx skills add danyuchn/asd-ste100-skill --all -g
```

Update both later with `npx skills update`. Restart the harness. Skills
load at session start.

### Manual install

If you want a git checkout you can edit and push from:

```bash
git clone https://github.com/manav03panchal/agent-harness-skill.git ~/.agents/skills/harness-engineering
ln -s ../../.agents/skills/harness-engineering ~/.claude/skills/harness-engineering
ln -s ~/.agents/skills/harness-engineering ~/.codex/skills/harness-engineering
```

Update with `git pull` in the skill directory.

## Companion skill

All text in this skill, and all text it produces, follows ASD-STE100. The
`asd-ste100` skill does the vetting. Install it beside this one (see
Install above).

## Layout

- `SKILL.md`: the map. Core beliefs, the 6-phase workflow, the diagnostic question.
- `templates/`: entry file, Claude shim, golden principles, PR loop, execution plan.
- `references/`: legacy audit, docs layout, architecture model, per-harness notes, enforcement ladder, content vetting.

## Sources

- https://openai.com/index/harness-engineering/
- https://github.com/danyuchn/asd-ste100-skill
- https://code.claude.com/docs/en/memory

## License

MIT.
