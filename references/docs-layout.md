# docs/ layout: the system of record

```
docs/
  ARCHITECTURE.md            # top-level map: domains, layers, package layout
  agent-operating-principles.md  # core beliefs about how agents work here
  golden-principles.md       # mechanical rules and how each one is enforced
  dependencies.md            # allowlist, reasons, candidates to wrap or replace
  domains/
    README.md                # index: domain, doc, owner, quality grade
    <domain>.md              # purpose, entities, layers, entry points, pitfalls
  design/
    README.md                # index with verification status per doc
    <topic>.md               # status: draft | verified | stale
  decisions/
    YYYY-MM-DD-<slug>.md     # context, decision, alternatives, consequences
  plans/
    TEMPLATE.md
    active/<slug>.md
    completed/<slug>.md
    debt.md                  # register: item, origin, severity, ratchet status
  quality/
    README.md                # grading rubric
    grades.md                # domain by layer grid. GC sweeps update it.
    harness-audit.md         # output of the audit checklist
  tooling/
    worktree-boot.md
    browser.md
    observability.md
    evidence.md
  process/
    pr-loop.md
    escalation.md
  style/
    content-vetting.md       # STE rule: what is vetted, in which mode (copy of the skill reference)
    ste-banned-words.txt     # phrasal verbs and marketing adjectives that the doc lint rejects
```

## Principles
- **Show detail progressively.** Entry file, then ARCHITECTURE, then the
  domain doc, then the design doc. Each level is small. Each level points
  to the next.
- **Put a status on everything that can rot.** Each design doc carries
  `status: verified | stale | draft` and `verified-against: <commit>`.
- **Keep plans and debt beside the code.** History is context. Agents read
  it instead of asking.
- **One fact, one place.** Cross-link. Never duplicate.

## Doc lint (add to CI)
- All relative links resolve.
- The entry file is 100 lines or fewer. Every path it names exists.
- Every `CLAUDE.md` (root and nested) starts with `@AGENTS.md`. A sibling
  `AGENTS.md` exists.
- Every directory under `src/<domain>` has a `docs/domains/<domain>.md`.
- Every design doc has `status` and `verified-against`.
- Every active plan changed within the last N days. Flag the others.
- `quality/grades.md` covers every domain in `domains/README.md`.
- The STE mechanical subset passes on every vetted file. See
  `references/content-vetting.md`. The subset is:
  - no semicolons
  - sentence length caps (20 Strict, 25 STE-flavored)
  - paragraphs of 6 sentences or fewer
  - no banned words from `style/ste-banned-words.txt`

## Doc-gardening sweep (scheduled)
- For each design doc marked `verified`: diff the code it describes since
  `verified-against`. If the code changed, mark the doc `stale` and open a
  PR to reconcile it.
- Find code paths with no owning domain doc. Open a PR that adds a stub.
- Find plans in `active/` whose PRs merged. Move them to `completed/`.
- Run the `asd-ste100` skill over vetted files that changed since the last
  sweep. Open a fix-up PR when the rewrite differs from the file.
