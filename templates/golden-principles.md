# Golden principles

These rules keep the codebase legible and consistent for future agent runs.
Each rule is opinionated and mechanical. Each rule has, or is scheduled to
get, a lint, a structural test, or a scheduled sweep. A rule without
enforcement is a wish.

Format: **Rule.** Why. How it is enforced.

## Boundaries
- **Parse, do not validate, at every boundary.** Parse external input into
  typed shapes at the edge. External input includes HTTP, database, files,
  environment, and third-party SDK responses. Internal code never rechecks
  the shape. Why: this stops guessed shapes from spreading. Enforced by:
  a lint that rejects raw `any` or untyped access outside `*/boundary/`
  modules.
- **Do not probe data "YOLO-style".** Never inspect a payload to guess its
  shape. Use the schema or the typed SDK. Why: guessed shapes compound into
  silent bugs. Enforced by: a review lens and a lint on dynamic key access
  in service layers.
- **Dependencies point forward only.**
  `Types → Config → Repo → Service → Runtime → UI`. Cross-cutting concerns
  enter through `Providers` only. Why: this keeps the dependency graph
  small enough to reason about. Enforced by: a structural test on the
  import graph.

## Reuse
- **Use shared utilities. Do not write local helpers.** If a helper would
  be the second of its kind, move it to the shared package. Why: this keeps
  invariants in one place. Enforced by: a sweep that finds near-duplicate
  helpers and opens a consolidation PR.
- **Prefer boring dependencies: composable and API-stable.** If you cannot
  understand a dependency's behavior from the repository, reimplement the
  subset you need. Add tests and instrumentation. Why: agents reason best
  over code they can read. Enforced by: the allowlist in
  `docs/dependencies.md` and a lint on new imports.

## Observability
- **Use structured logging only.** Key-value fields. No string
  interpolation. Consistent field names. Why: agents can query structured
  logs. Enforced by: a lint.
- **Every user journey has spans.** Trace critical paths from start to end.
  Why: this makes "no span over 2 s" a testable prompt. Enforced by: a
  structural test that maps routes to spans.

## Shape
- **File size limit: N lines.** Choose N once. Why: each unit stays inside
  the context window. Enforced by: a lint.
- **Naming conventions** for schemas, types, handlers, and tests. Document
  them here. Why: predictable names beat clever names. Enforced by: a lint.
- **Tests live beside the unit they test.** They mirror its name. Each test
  runs in isolation. Why: agents find and extend them. Enforced by: a lint.

## Docs
- **Vet all agent-facing text with ASD-STE100.** This covers the entry
  file, docs, lint and hook messages, tool descriptions, plans, and PR
  text. Run the `asd-ste100` skill before commit. Use Strict mode for rules
  and procedures. Use STE-flavored mode for descriptive prose. Why: agents
  cannot ask what you meant. Enforced by: the doc lint checks the
  mechanical subset (no semicolons, sentence length, banned phrasal verbs).
  The `clarity` review lens runs the full skill. See
  `docs/style/content-vetting.md`.
- **If a discussion aligned the team, write the result in `docs/`.** Why:
  text outside the repository is invisible to agents. Enforced by: the
  doc-gardening sweep flags code with no owning design doc.
- **Plans are artifacts.** Non-trivial work gets an execution plan with a
  decision log. Commit the plan. Why: history is context. Enforced by: the
  doc lint.

## Process
- **Lint messages teach.** Every custom lint error includes the fix. Why:
  the message is a prompt. Enforced by: a test on the lint's own output.
- **Small PRs with short lives.** Why: corrections are cheap and waiting is
  expensive. Enforced by: a sweep that flags PRs over N files or N days
  old.
