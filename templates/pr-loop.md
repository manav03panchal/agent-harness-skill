# PR completion loop

Run this loop from any harness. Do not stop early. Hand the work to a human
only at step 7 or when an escalation condition applies.

## Loop
1. **Review your own diff.** Compare it with `docs/golden-principles.md`
   and the domain's architecture doc. Fix what you find before anyone else
   sees it. Run the `asd-ste100` skill on every changed doc, lint message,
   hook message, and tool description. Run it on the PR title and
   description too. Use Strict mode for rules, procedures, and messages.
   Use STE-flavored mode for prose.
2. **Run the full local gate:** format, lint, typecheck, unit tests,
   structural tests, doc lint. Fix every failure. The lint messages tell
   you how.
3. **Verify behavior, not only tests.** Boot the worktree instance. Drive
   the UI or call the API. Query the logs and metrics. For bugs: capture
   before and after evidence (screenshot, video, or log excerpt). Attach
   the evidence to the PR.
4. **Open the PR.** Include: the intent, a link to the execution plan, what
   you verified and how, the evidence, and known gaps.
5. **Request agent reviews.** Each reviewer gets fresh context and one
   lens: correctness, architecture and layering, security, docs-drift, test
   adequacy, or clarity. The clarity reviewer runs `asd-ste100` over the
   changed text. It reports each violation with the rule name and the
   rewrite. Use what your harness offers: subagents, `/review`, a cloud
   review, or a second CLI session. The lens matters. The tool does not.
6. **Answer every comment.** Fix the issue, or reply with your reasoning.
   Request review again. Repeat until every reviewer is satisfied and CI is
   green. If CI fails, diagnose the failure and fix it yourself.
7. **Merge** with squash. Then update `docs/`: mark the plan completed,
   record decisions, and adjust quality grades if you changed them.

## Escalate to a human only if
- A product or UX tradeoff has no documented principle that resolves it.
- The action is irreversible or outward-facing, and no one pre-authorized
  it. Examples: a data migration, an external send, a production config
  change.
- Two reviewers disagree and the docs do not settle the disagreement.
- You looped 3 times on the same failure with no progress.

When you escalate, state: the decision you need, the options, your
recommendation, and what is blocked. After the human answers, write the
decision in `docs/decisions/`. Then nobody needs to ask it again.
