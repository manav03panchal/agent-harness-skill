# Architecture model: enforce the boundaries, free the interior

This is the default shape. Adapt the names to the codebase. Keep the three
properties: **fixed layers, forward-only dependencies, one door for
cross-cutting concerns.**

```
Per business domain (for example billing, settings, search):

  Types → Config → Repo → Service → Runtime → UI
    ▲
    └── Providers (auth, telemetry, feature flags, connectors, clock, ids)
        enter at any layer through ONE explicit interface. Nothing else
        crosses domains or points backward.
```

| Layer   | Holds                                       | May import          |
|---------|---------------------------------------------|---------------------|
| Types   | schemas, domain types, parsers              | nothing in-domain   |
| Config  | typed config, defaults                      | Types               |
| Repo    | persistence, external I/O, boundary parsing | Types, Config       |
| Service | business logic, pure where possible         | Types, Config, Repo |
| Runtime | wiring, jobs, handlers, lifecycle           | Service and below   |
| UI      | presentation                                | Runtime and below   |

A cross-domain call goes through the other domain's Service. It uses
Providers or an explicit public interface. It never imports the other
domain's Repo or internals.

## Why this early
Teams usually defer this structure until they have hundreds of engineers.
With agents it is a prerequisite. The constraints let throughput rise
without drift.

## Enforcement (the agent writes all of it)
- **Import-graph structural test.** Parse the imports. Assert that every
  edge is in the allowed set. On failure, report: the file, the offending
  import, the allowed set, and the fix. Example fix text: "Move this to
  Service and call it from Runtime." Or: "Expose this through Providers."
- **Boundary lint.** Outside `Repo` and `Types`, reject untyped access to
  external data. Parse at the edge (parse, do not validate).
- **Taste lints.** Structured logging. Naming. File size. No local helper
  when a shared one exists. Platform reliability rules: timeouts, retries
  with jitter, idempotency keys where relevant.
- **Every message contains the fix.** Test the lint's output text. The text
  is a prompt.

## Legacy ratchet
1. Write the structural test. Run it. Count the violations. Record the
   count as the baseline in `docs/plans/debt.md`.
2. Gate: violations in touched files must not increase. New files must
   have zero violations.
3. The GC sweep opens one-violation PRs until the baseline reaches zero.
4. Then switch the gate to hard-fail.

## What NOT to constrain
Do not constrain the library choice inside a layer, the internal function
structure, or style beyond the formatter. When the boundary is clear, the
agent picks reasonable tools. Example: it reaches for a schema library at
the boundary without an instruction to use a specific one. Judge the output
on three questions. Is it correct? Is it maintainable? Can the next agent
run read it? Do not judge it on whether a human would write it that way.
