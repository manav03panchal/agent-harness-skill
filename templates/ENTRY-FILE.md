<!--
Template for the canonical agent entry file. Save it as AGENTS.md.

Other harnesses get a MECHANICAL shim to this file. Never use a prose pointer.
  Claude Code  → CLAUDE.md. Line 1 is `@AGENTS.md`. See templates/CLAUDE.md.
                 Claude does NOT read AGENTS.md. The @import inlines it at launch.
  Gemini CLI   → GEMINI.md with the same @AGENTS.md import.
  Cursor       → reads AGENTS.md. Also reads .cursor/rules/*.mdc for scoped rules.
  Copilot      → reads AGENTS.md.
Verify that each harness loaded the file. Claude: run /context and read Memory files.

Rules for this file:
- 100 lines or fewer. A CI lint enforces this.
- Pointers, not prose. Text longer than 3 lines belongs in docs/.
- Every path in this file must exist. A lint checks this.
- Write it in Strict ASD-STE100. Vet it with the asd-ste100 skill.
-->

# <Project name>: agent entry point

Coding agents operate this repository. This file is a **map**. Follow the
links before you act. Do not guess.

## Start here
- Architecture overview: `docs/ARCHITECTURE.md`
- Domain index (what exists, where, owner, quality grade): `docs/domains/README.md`
- Golden principles (non-negotiable, enforced by tooling): `docs/golden-principles.md`
- Core beliefs about how we work with agents: `docs/agent-operating-principles.md`

## Before you change anything
1. Create a worktree for the task: `git worktree add ../<repo>-<slug> -b <slug>`.
   Work there. Never edit files on `main`. In Claude Code, the EnterWorktree
   tool does this in one step.
2. Read the domain doc for the area you will touch.
3. Check `docs/plans/active/` for a plan that overlaps your task.
4. For non-trivial work, create an execution plan from `docs/plans/TEMPLATE.md`.

## Commands
- Install dependencies: `<cmd>`
- Boot an isolated instance for this worktree: `<cmd>` (see `docs/tooling/worktree-boot.md`)
- Test: `<cmd>`
- Lint: `<cmd>`
- Typecheck: `<cmd>`
- Architecture and structure tests: `<cmd>`
- Doc lint: `<cmd>`

## Verify your own work
- Drive the UI: `docs/tooling/browser.md`
- Query logs, metrics, and traces for this worktree: `docs/tooling/observability.md`
- Record before and after evidence for UI bugs: `docs/tooling/evidence.md`

## Shipping
- PR completion loop (self-review, agent reviews, iterate, merge): `docs/process/pr-loop.md`
- Escalate to a human only when a decision needs judgment: `docs/process/escalation.md`
- Lint errors contain the fix. Do what the message says.

## Writing text that agents will read
- Docs, messages, descriptions, and PR text follow ASD-STE100.
- Vet the text with the `asd-ste100` skill before commit.
- Rules and modes: `docs/style/content-vetting.md`

## Where knowledge lives
- Design docs with verification status: `docs/design/`
- Decision log: `docs/decisions/`
- Quality grades per domain and layer: `docs/quality/`
- Known technical debt: `docs/plans/debt.md`
- Completed plans (history): `docs/plans/completed/`

## Prohibited actions
- Do not store knowledge outside the repository. If it matters, write it in `docs/`.
- Do not bypass a lint. Fix the cause, or open a debt entry with a reason.
- Do not add a dependency before you read `docs/dependencies.md`.
