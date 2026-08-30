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

User level, shared by every harness on the machine:

```bash
git clone git@github.com:manav03panchal/agent-harness-skill.git ~/.agents/skills/harness-engineering
ln -s ../../.agents/skills/harness-engineering ~/.claude/skills/harness-engineering
ln -s ~/.agents/skills/harness-engineering ~/.codex/skills/harness-engineering
```

Repository level, shared by the team:

```bash
git clone git@github.com:manav03panchal/agent-harness-skill.git .agents/skills/harness-engineering
ln -s ../../.agents/skills/harness-engineering .claude/skills/harness-engineering
```

Update with `git pull` in the skill directory. Restart the harness. Skills
load at session start.

## Companion skill

All text in this skill, and all text it produces, follows ASD-STE100. Install
the vetting skill beside it:

```bash
git clone https://github.com/danyuchn/asd-ste100-skill ~/.agents/skills/asd-ste100
```

## Layout

- `SKILL.md`: the map. Core beliefs, the 6-phase workflow, the diagnostic question.
- `templates/`: entry file, Claude shim, golden principles, PR loop, execution plan.
- `references/`: legacy audit, docs layout, architecture model, per-harness notes, enforcement ladder, content vetting.

## Sources

- https://openai.com/index/harness-engineering/
- https://github.com/danyuchn/asd-ste100-skill
- https://code.claude.com/docs/en/memory
