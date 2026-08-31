# Enforcement ladder: when prose is not enough

Every harness delivers the entry file as context. No harness treats it as
policy. Claude's docs say: "Claude treats them as context, not enforced
configuration. To block an action regardless of what Claude decides, use a
PreToolUse hook." Codex behaves the same way. So decide, per rule, which
rung it needs. Do not stop at the bottom rung for a rule that must hold.

| Rung | Mechanism | Holds when | Harness-specific? |
|------|-----------|------------|-------------------|
| 1 | Entry file or docs prose | The agent reads it and agrees | No |
| 2 | Formatter, lint, or structural test, run locally | The agent runs the gate | No |
| 3 | The same gate as a required CI check | Always, before merge | No |
| 4 | A harness hook that runs the gate on every edit or commit | Always, during the session | Yes |
| 5 | A harness permission deny or a sandbox | Always. The agent cannot reason around it. | Yes |

Rules of thumb:
- Architecture, boundaries, naming, logging, file size: use rung 3 (a
  custom lint in CI, with the fix in the message). Add rung 4 so the agent
  gets the feedback at edit time, not at PR time.
- "Never push to main", "never run migrations", "never touch production
  config": use rung 5. Prose alone is negligent here.
- Style preferences, tone, "prefer X": rung 1 is enough.

## Rung 4: hooks

Keep the logic in a harness-neutral script, for example
`scripts/gate.sh <file>`. Make each harness hook a one-line call to that
script. Then every driver gets the same gate and the same messages.

A worked example for rung 4: the worktree rule. The entry file says
"create a worktree for the task". The hook makes the rule hold. A
`PreToolUse` hook on `Edit|Write` runs `scripts/guard-branch.sh`. The
script exits with code 2 when the current branch is `main`. Its stderr
says: "You are on main. Create a worktree first: git worktree add
../<repo>-<slug> -b <slug>. Then continue there."

**Claude Code.** Commit `.claude/settings.json` so the team shares it:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [{ "type": "command", "command": "scripts/gate.sh \"$CLAUDE_FILE_PATH\"" }]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{ "type": "command", "command": "scripts/guard-bash.sh" }]
      }
    ]
  }
}
```
A hook that exits with code 2 blocks the action. Claude Code feeds the
hook's stderr back to the model. Write that stderr as fix instructions, in
the same way as a lint message. If you suspect that a shim did not load,
use the `InstructionsLoaded` hook to log each instruction file load.

**Codex.** Use `hooks.json` in the repository or in `~/.codex/hooks.json`.
Bind the equivalent pre-tool and post-tool events to the same scripts.
Check the docs for the installed version for the current event names.

**Other harnesses.** If the harness has no hook surface, rung 3 (CI) is the
floor. A git pre-commit hook (`.githooks/pre-commit` → `scripts/gate.sh`)
is a reasonable rung 4 substitute, because agents commit through git.

## Rung 5: hard denies

- Claude Code: `permissions.deny` in `.claude/settings.json`. Examples:
  deny `Bash(git push --force*)` and `Edit(infra/prod/**)`. Use managed
  settings for organization-wide policy.
- Codex: the sandbox and approval policy in `config.toml`.
- All harnesses: branch protection and required checks on the remote. Then
  a harness with no local enforcement still cannot merge around the gate.

## Verify that the harness sees what you expect

- Claude: run `/context` and read Memory files. The list must include
  `CLAUDE.md`. After Claude reads a nested file, the list must include the
  nested one. `/doctor` flags oversized files.
- Codex: start a session. Ask it to print the instructions it loaded.
- Add a CI check. It fails if any `CLAUDE.md` does not start with
  `@AGENTS.md` or lacks a sibling `AGENTS.md`.
