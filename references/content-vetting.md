# Content vetting: ASD-STE100

Vet all agent-facing English in the repository with ASD-STE100 Simplified
Technical English before commit. Agents read this text with no human to
ask "did you mean X or Y?". STE removes the two causes of misreading: words
with more than one meaning, and sentences with more than one structure.

Vetting uses the `asd-ste100` skill:
https://github.com/danyuchn/asd-ste100-skill

Install it at repository level: `npx skills add danyuchn/asd-ste100-skill`.
Or install it at user level:
`git clone https://github.com/danyuchn/asd-ste100-skill ~/.agents/skills/asd-ste100`.
Then symlink it into `.claude/skills/` and `.codex/skills/` as needed.

## What is vetted, and in which mode

| Content | Mode | Why |
|---|---|---|
| Entry file (`AGENTS.md`) and shims | Strict | Every session loads it. One misread rule repeats on every task. |
| `docs/golden-principles.md` | Strict | A rule must have one reading. |
| Custom lint messages and hook stderr | Strict | The harness injects them into agent context as fix instructions. |
| Tool, skill, and script descriptions | Strict | The agent parses them to decide when to call the tool. |
| Escalation messages to humans | Strict | The human decides from this text alone. |
| `docs/process/*`, `docs/tooling/*` | Strict | These are procedures. One instruction per sentence. |
| `docs/domains/*`, `docs/design/*`, `docs/ARCHITECTURE.md` | STE-flavored | Descriptive prose. Structural rules apply. The vocabulary lockdown does not. |
| Execution plans and decision logs | STE-flavored | Future agents read them as context. |
| PR titles and descriptions | STE-flavored | Review agents read them. |
| Code comments that state a rule or a constraint | Strict | Same as lint messages. |
| Commit messages | STE-flavored | History is context. |

Not vetted: user-facing UI copy, marketing text, and any text where voice
is the point. STE is flat by design.

## How to apply it

1. Write the text.
2. Run the `asd-ste100` skill on it. State the mode. Ask for the diff only
   when you want to learn from the changes.
3. Use the output. If the skill adds a `Kept as-is:` line, keep that
   phrasing. The skill kept it because a shorter form would lose a
   condition, a number, or a hedge.
4. Do not let the rewrite drop a fact, a scope qualifier, or a hedge. A
   rewrite that changes the claim is not a rewrite.

Run the check on every changed file of a type in the table above before
you open the PR. Review agents run it again as the `clarity` lens.

## Mechanical subset: enforce it in the doc lint

The `asd-ste100` skill does the full check. The doc lint enforces the
subset that a script can check without judgment. Then a violation fails CI
even when nobody runs the skill:

- No semicolons in any vetted file.
- Sentence length: 20 words or fewer in Strict files. 25 words or fewer in
  STE-flavored files. Allow an escape comment on the line before:
  `<!-- ste: long-sentence <reason> -->`.
- Paragraphs of 6 sentences or fewer.
- Noun clusters of 3 words or fewer. Approximate this as four or more
  consecutive nouns. Report it as a warning.
- No banned phrasal verbs or marketing adjectives. Keep the list in
  `docs/style/ste-banned-words.txt`. Start with: spin up, reach out, dive
  into, kick off, set up (as a verb), seamless, robust, powerful,
  effortless, cutting-edge, blazing-fast. Extend the list when reviews
  find more.
- Lint messages must pass the lint themselves. Test the lint's output.

The lint error message names the file, the line, the rule, and the fix.
Example:
`docs/process/pr-loop.md:14: sentence has 27 words (max 20 in Strict).
Split it into two sentences. One instruction per sentence.`

## Where the rule lives in the repository

- `docs/golden-principles.md`, section "Docs", principle "Vet all
  agent-facing text with ASD-STE100."
- `docs/process/pr-loop.md`, step 1 (self-review) and step 5 (the
  `clarity` review lens).
- The doc lint in CI enforces the mechanical subset above.
- The doc-gardening sweep runs the skill over files that changed since the
  last sweep. It opens a fix-up PR when the rewrite differs.
