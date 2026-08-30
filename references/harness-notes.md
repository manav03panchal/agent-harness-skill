# Harness-specific notes

The approach is the same in every harness. Three things differ: the
entry-file name, how the harness starts sub-reviews, and how scheduled
sweeps run.

| Harness | Entry file(s) read | Nested or scoped files | Skills dir | Sub-agent review | Scheduled sweeps |
|---------|--------------------|------------------------|------------|------------------|------------------|
| Claude Code | `CLAUDE.md` **only**. It reaches `AGENTS.md` through the `@AGENTS.md` import shim. | Nested `CLAUDE.md` (loads on demand). `.claude/rules/*.md` with `paths:`. | `.claude/skills/`, `.claude/commands/` | `Agent` tool, `/code-review` | cron or CI that runs `claude -p`. Scheduled cloud routines. |
| OpenAI Codex | `AGENTS.md` | Nested `AGENTS.md` | repo skills folder | cloud reviews, a second run | Codex cloud tasks, CI |
| Cursor | `.cursor/rules/*.mdc`, `AGENTS.md` | rule `globs` | none | background agents | CI |
| GitHub Copilot | `.github/copilot-instructions.md`, `AGENTS.md` | `.github/instructions/*.instructions.md` | none | Copilot code review | Actions |
| Gemini CLI | `GEMINI.md` | Nested `GEMINI.md` | none | a second session | CI |
| Aider and others | `CONVENTIONS.md` or configured | none | none | none | CI |

## Claude Code specifics (checked against code.claude.com/docs/en/memory)
- **Claude Code reads `CLAUDE.md` only. It never reads `AGENTS.md`.** The
  docs say: "Claude Code reads `CLAUDE.md`, not `AGENTS.md`."
- **The shim is the `@AGENTS.md` import.** Claude Code expands it and
  inlines it at launch. Imports nest up to 4 hops deep. Paths resolve
  relative to the importing file. A prose sentence "see AGENTS.md" is not
  an import. `@AGENTS.md` inside backticks or a fenced block is not an
  import either. The parser skips code. A symlink `CLAUDE.md → AGENTS.md`
  also works. On Windows a symlink needs admin rights, so prefer the
  import.
- Claude Code loads `CLAUDE.md` from the working directory and every
  ancestor at launch. **Nested** `CLAUDE.md` files in subdirectories load
  on demand, when Claude reads files there. So every nested `AGENTS.md`
  needs a nested `CLAUDE.md` shim beside it.
- `.claude/rules/*.md` load like CLAUDE.md. With `paths:` frontmatter they
  load only when Claude touches matching files. Use them for path-scoped
  rules instead of a large root file. Keep them thin. Point them at
  `docs/` too. Otherwise the knowledge is Claude-only.
- Target under 200 loaded lines in total. Imports count toward context.
- Claude strips HTML comments before injection. Use them for free
  maintainer notes.
- The root `CLAUDE.md` survives `/compact`. Claude reads it from disk
  again. Nested files and path rules reload only when Claude reads a
  matching file again.
- **The file is context, not enforcement.** For rules that must hold, use
  hooks (`PreToolUse`, `PostToolUse` in `.claude/settings.json`) or
  `permissions.deny`. See `references/enforcement.md`.
- Verify: run `/context` and read Memory files. The `InstructionsLoaded`
  hook logs each instruction file load with the reason.
- `/import` (v2.1.213 and later) copies `AGENTS.md` into `CLAUDE.md` one
  time. **Do not use it here.** A copy drifts. An import does not.

## Making one repository serve all harnesses
- Make `AGENTS.md` canonical. It has the widest native support: Codex,
  Cursor, Copilot, and others. Claude and Gemini get an `@AGENTS.md` import
  shim (`templates/CLAUDE.md`). Lint that every shim is intact.
- Keep skills and scripts harness-neutral. Put plain scripts under
  `scripts/` or `tools/`. Document them in `docs/tooling/`. Make each
  harness-specific skill folder a thin wrapper that calls those scripts.
- Lints and structural tests live in the repository and run in CI. They do
  not care which agent produced the diff.
- The PR loop (`templates/pr-loop.md`) names lenses and steps, not tools.
  Map each step to what the harness offers.

## Companion skill: asd-ste100
Install it beside this skill in every harness. Use the same folder layout
and the same symlink pattern. Repository level:
`npx skills add danyuchn/asd-ste100-skill`. User level: clone to
`~/.agents/skills/asd-ste100`. Then symlink into `~/.claude/skills/` and
`~/.codex/skills/`.
