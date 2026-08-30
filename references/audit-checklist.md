# Legacy repository audit

Goal: find what makes agents fail in this repository. Rank the causes by how
often they cause failure. Write the result to `docs/quality/harness-audit.md`.
Give each area a grade (A to F), evidence, and the top 3 fixes. Then put the
fixes into an execution plan.

Score each question: 0 (absent), 1 (partial), 2 (solid).

## 1. Entry file and map
- Does an entry file exist for each harness in use? Is one file canonical?
- Is the file 100 lines or fewer? Is it mostly pointers?
- Does every path in the file exist?
- Does the file say how to install, run, test, lint, and verify?
- Is any content stale? Does any content contradict the code?

## 2. Knowledge base
- Is there a `docs/` directory (or equivalent)? Could an agent learn the
  full business domain from it without a human?
- Is there an architecture map with domains and layers?
- Does each domain have a doc with an owner and a quality grade?
- Do design docs carry a verification status?
- Is there a decision log? A plan history? A debt register?
- Does a lint check any of this (links, freshness, structure)?
- Where does tribal knowledge live today? Ask the team. List each source
  that the team must move into the repository.

## 3. Architecture enforcement
- Does any doc define the layers and boundaries?
- Does a tool enforce them (import lint, structural test)? Or only
  convention?
- Do the lint and test error messages explain how to fix the violation?
- Does the code parse data at boundaries? Or does it pass untyped data
  around?
- Count the violations of the intended layering. This is the baseline for
  the ratchet.

## 4. Verification loop
- Can one command install everything from a clean clone? Does it work?
- Can the app boot in isolation per branch or worktree (ports, database,
  fixtures)?
- Can an agent drive the UI? Is browser automation available and
  documented?
- Test suite: how long does it run? What is the flake rate? What does it
  not cover?
- Can an agent check non-functional requirements locally (latency, memory,
  error rate)?

## 5. Observability legibility
- Are the logs structured? Are the field names consistent?
- Do critical journeys have traces?
- Is there a local, ephemeral stack that an agent can query? Are query
  examples documented?

## 6. Review and merge process
- What blocks a merge today? Which gates carry load? Which are ritual?
- What are the median PR age and size?
- Does an agent perform any review? Can an agent answer review comments and
  push fixes itself?
- Are the escalation criteria written down?

## 7. Entropy and cleanup
- Sample 20 files. Count duplicate helpers, inconsistent patterns, and dead
  code.
- Is there a recurring cleanup process? Is it manual or scheduled?
- Are golden principles written down? Does a tool enforce them?

## 8. Content clarity (ASD-STE100)
- Sample the entry file, 5 docs, 10 lint or error messages, and 5 tool or
  script descriptions. Run the `asd-ste100` skill with "show the diff" on
  each sample.
- Count violations per 100 sentences. Count each of these:
  - semicolons
  - sentences over the cap
  - passive voice with no actor
  - phrasal verbs
  - several names for one concept
  - hedge stacks
- Does a lint check any of this today?
- Note the worst offenders. Fix the entry file and lint messages first.
  Every session reads them.

## 9. Dependencies
- List dependencies that you cannot understand from the repository:
  opaque, unstable API, or poorly documented. These are candidates to wrap
  or replace.
- Is there an allowlist or a policy for new dependencies?

## Output
```
# Harness audit: <repo>, <date>
| Area | Grade | Top failure mode | Fix 1 | Fix 2 | Fix 3 |
...
## Sources of tribal knowledge to move into the repository
## Violation baseline (for the ratchet)
## Recommended phase order
```
