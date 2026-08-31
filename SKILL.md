---
name: harness-engineering
description: Rework a codebase (greenfield or legacy) so coding agents can do reliable, high-throughput work in it. Treats the repository as the agent's environment - map-style entry file, docs/ as system of record, mechanically enforced architecture, agent-legible app and logs, continuous garbage collection. Use when asked to "make this repo agent-ready", "set up harness engineering", "improve agent legibility", "onboard agents to this codebase", "write an AGENTS.md/CLAUDE.md", "reduce AI slop", or when an agent keeps failing at tasks in a repo because the environment is underspecified. Harness-agnostic - works for Claude Code, Codex, Cursor, Copilot, Gemini CLI, or any tool that reads a repo-local instruction file.
---

# Harness Engineering

The premise: when agents write the code, the engineer designs the
environment. The environment is the tools, the constraints, the docs, and
the feedback loops. A good environment lets agents do reliable work. When an
agent fails, do not tell it to "try harder". Ask: what capability is
missing? Then make that capability legible and enforceable.

Human time and attention are the scarce resource. Every rule below exists to
spend less of it per unit of shipped, correct software.

## Core beliefs (apply everywhere)

1. **If it is not in the repository, it does not exist.** Agents cannot see
   Slack threads, Google Docs, or people's heads. Put knowledge into
   versioned files in the repository: code, markdown, schemas, plans.
2. **Map, not manual.** The entry file (`AGENTS.md`, `CLAUDE.md`,
   `.cursorrules`, `GEMINI.md`) is a table of contents of about 100 lines.
   Depth lives in `docs/`. Show detail progressively. Do not load it all
   at the start.
3. **Enforce invariants, not implementations.** Encode boundaries, layering,
   and taste as linters and structural tests. Each error message must
   contain the fix. Inside those limits, leave the agent free.
4. **Legibility over style.** Make the codebase one that an agent can reason
   about from the repository alone. Prefer "boring" technology: composable,
   API-stable, well documented. A small reimplementation in the repository
   can beat an opaque dependency.
5. **Corrections are cheap. Waiting is expensive.** Keep PRs short-lived.
   Keep blocking gates minimal. Re-run a flaky test instead of blocking on
   it. This is safe only when throughput is high and verification is
   automated. Build that first.
6. **Collect garbage continuously.** Agents copy the patterns that exist,
   including bad ones. Encode golden principles. Then run recurring cleanup
   tasks that open small PRs. A reviewer must read each PR in under one
   minute.
7. **Every fix goes back into the repository.** Review comments, refactor
   PRs, and user bugs become doc updates or tooling. When docs are not
   enough, promote the rule into code.
8. **Vet all agent-facing text with ASD-STE100.** This covers entry files,
   docs, lint messages, hook output, tool descriptions, plans, and PR text.
   Run all of it through the `asd-ste100` skill before commit. Agents
   cannot ask what you meant. One meaning per word. One instruction per sentence. Active
   voice. No semicolons. Rules and modes: `references/content-vetting.md`.

## Workflow

Run the phases in order. Each phase unlocks the next. Do not skip to
autonomy.

### Phase 0: Assess (legacy) or Scaffold (greenfield)

**Legacy:**
1. Run the audit in `references/audit-checklist.md`.
2. Write `docs/quality/harness-audit.md`. Grade each area: entry file,
   docs, architecture enforcement, verification loop, observability,
   content clarity, cleanup.
3. Rank the gaps by how often they cause agent failure. Do not rank by
   size.

**Greenfield:**
1. Scaffold the repository structure, CI, formatter, package manager, and
   entry file.
2. Have the agent generate all of it. Use the templates in `templates/`.
3. Commit the empty but structured repository first.

### Phase 1: Knowledge base as the system of record

- Write or rewrite the entry file from `templates/ENTRY-FILE.md`. Keep it
  at 100 lines or fewer. Use pointers, not prose.
- **Mixed-harness teams (Claude, Codex, and others): `AGENTS.md` is
  canonical. Every other harness gets a mechanical shim, never a prose
  pointer.** Claude Code does not read `AGENTS.md`. It reads only
  `CLAUDE.md`. The sentence "See AGENTS.md" is advice that the model can
  ignore. The line `@AGENTS.md` is an import that Claude Code inlines at
  launch. Use `templates/CLAUDE.md` without changes. Line 1 is
  `@AGENTS.md`, not in backticks, not in a code block. Add one shim in each
  directory that has a nested `AGENTS.md`. Then verify: start `claude` in
  the repository and run `/context`. Confirm that Memory files lists
  `CLAUDE.md` with the imported content. Per-harness rules:
  `references/harness-notes.md`.
- **Prose is not enforcement, in any harness.** A rule that must always
  hold goes into a lint, a structural test, a CI gate, or a harness hook. Claude uses `PreToolUse` and `PostToolUse` hooks. Codex uses
  `hooks.json`. The entry file tells the agent where the rules are. The
  tooling makes the rules impossible to skip. See
  `references/enforcement.md`.
- Create `docs/` per `references/docs-layout.md`. It holds the
  architecture map, the domain index, design docs with verification
  status, quality grades, plans (active, completed, debt), and the golden
  principles.
- Vet every file you write in this phase with the `asd-ste100` skill. Use
  Strict mode for the entry file, golden principles, and process docs. Use
  STE-flavored mode for architecture, domain, and design docs. If the
  skill is missing, install it: `npx skills add danyuchn/asd-ste100-skill`.
- Add a doc linter to CI. It checks:
  - All cross-links resolve.
  - Every domain has an entry.
  - Every design doc has a status.
  - The entry file is under the line limit.
  - The mechanical STE subset from `references/content-vetting.md` passes.
  - Every `CLAUDE.md` (root and nested) starts with `@AGENTS.md` and has a
    sibling `AGENTS.md`. This check stops the two files from drifting
    apart.
- Schedule a doc-gardening task (Phase 5). It compares docs with real
  behavior and opens fix-up PRs.

### Phase 2: Mechanically enforced architecture

- Define the layers per domain and the permitted dependency edges. The
  default model is in `references/architecture-model.md`:
  `Types → Config → Repo → Service → Runtime → UI`. Cross-cutting concerns
  enter through one `Providers` interface. Adapt the names to the
  codebase. Keep the shape: forward-only, fixed layers, one door for
  cross-cutting concerns.
- Write custom lints and structural tests for:
  - layer direction
  - boundary parsing (parse, do not validate, at every I/O edge)
  - structured logging
  - naming conventions
  - file size limits
  - platform reliability rules
- **Every lint error message must say how to fix the error.** The harness
  injects the message into agent context. Treat it as a small prompt.
  Write it in Strict STE and vet it with the `asd-ste100` skill.
- Legacy repositories: enforce on new and touched code first (the ratchet).
  Record the count of existing violations as debt in
  `docs/plans/debt.md`. Let the GC loop reduce the count.

### Phase 3: Verification the agent can drive

After throughput, the next bottleneck is human QA. Remove humans from the
verification path wherever a machine can replace them.

- Require a worktree per task. The entry file states the rule. A hook
  enforces it: block edits while the session is on `main`
  (`references/enforcement.md`). Agents then start every task with
  `git worktree add`, and cleanup is one `git worktree remove`.
- Make the app bootable per worktree with one command. Isolate ports and
  databases. Tear the instance down after the task.
- Expose the UI to the agent through browser automation (CDP, Playwright,
  or `agent-browser`). Add skills for screenshots, DOM snapshots, and
  navigation. Add a skill that records a video of the bug and of the fix.
- Expose observability locally: ephemeral logs, metrics, and traces per
  worktree. Make them queryable (LogQL, PromQL, or an equivalent). This
  makes prompts such as "startup under 800 ms" or "no span in journey X
  over 2 s" testable.
- Document all of it in `docs/tooling/`. Point to it from the entry file.

### Phase 4: Agent-to-agent review loop

Encode the PR loop so the human reads only escalations:

1. The agent implements the change. Then it reviews its own diff against
   the golden principles and the architecture docs.
2. The agent requests independent agent reviews. Each reviewer has fresh
   context and one lens: correctness, architecture, security, docs-drift,
   or clarity. The clarity reviewer runs `asd-ste100` over changed docs,
   messages, and descriptions.
3. The agent answers every comment. It iterates until all reviewers are
   satisfied.
4. The agent detects CI failures and fixes them.
5. The agent escalates to a human only when a decision needs judgment.
   Examples: a product tradeoff, an irreversible action, an ambiguous spec.
6. The agent merges.

Save this loop as a repository-local skill or script. Then it runs the same
way from any harness. Template: `templates/pr-loop.md`.

### Phase 5: Garbage collection

- Write `docs/golden-principles.md` from `templates/golden-principles.md`.
  Keep the rules opinionated, mechanical, and short. Examples: prefer
  shared utilities over hand-rolled helpers. Do not probe data
  "YOLO-style". Validate at boundaries or use typed SDKs.
- Run recurring background tasks (cron, a CI schedule, or a scheduled
  agent). They scan for deviations, update `docs/quality/` grades, and open
  targeted refactor PRs. Size each PR so a reviewer reads it in under one
  minute. Automerge when CI is green.
- Run doc-gardening on the same schedule.
- Success metric: humans no longer need a "cleanup day".

### Phase 6: Increase autonomy deliberately

Trust the harness with end-to-end feature work only after Phases 1 to 5
hold. End-to-end means: validate, reproduce, record, fix, validate, record,
open the PR, respond, fix failures, escalate, merge. Autonomy is a result of
the environment. It is not a setting.

## When the agent struggles: the diagnostic question

Do not retry with a longer prompt. Ask these questions in order:

1. **Is context missing?** Add a doc. Link it from the map.
2. **Is a tool missing?** Add a script or skill. Document how to call it.
3. **Is a constraint missing?** Add a lint. Put the fix in the message.
4. **Is feedback missing?** Make the outcome observable: a test, a log, a
   metric, a screenshot. Then the agent can check its own work.
5. **Does the task need judgment?** That is the human's job. Record the
   decision in `docs/decisions/` afterward.

Then have the agent write the fix.

## Anti-patterns

- An entry file of 1,000 lines. Context is scarce. When everything is
  important, nothing is. The file rots. Nobody can lint it.
- Style rules in prose. If a rule matters, make it a lint.
- Knowledge in chat. If a discussion aligned the team, write the result in
  `docs/`.
- Blocking merges on flaky tests in a high-throughput system.
- A manual weekly cleanup of AI slop. Encode the principle. Schedule the
  sweep.
- Human-written "quick fixes" that bypass the harness. They teach the agent
  nothing. The agent will copy the surrounding pattern anyway.

## Files in this skill

- `templates/ENTRY-FILE.md`: the map of about 100 lines. Save it as `AGENTS.md`.
- `templates/CLAUDE.md`: the Claude Code shim (an import, not a pointer).
- `templates/golden-principles.md`: starter rules with enforcement.
- `templates/pr-loop.md`: the agent-driven PR completion loop.
- `templates/execution-plan.md`: a plan with progress and decision logs.
- `references/audit-checklist.md`: legacy repository assessment.
- `references/docs-layout.md`: `docs/` structure and lint rules.
- `references/architecture-model.md`: layering and enforcement.
- `references/harness-notes.md`: entry-file names and behavior per harness.
- `references/enforcement.md`: lints, CI gates, and hooks. Which to use when.
- `references/content-vetting.md`: the ASD-STE100 rule. What is vetted, which mode, the lint subset.
