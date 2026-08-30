@AGENTS.md

<!--
Do not add project instructions to this file. AGENTS.md is canonical. This
file only imports it. Claude Code reads CLAUDE.md, not AGENTS.md. The
import gives Claude Code the same instructions as every other harness.

Rules for this file. The doc lint enforces them.
- Line 1 must be exactly `@AGENTS.md`.
- Do not put the import in backticks or in a code block. Claude's import
  parser skips those. The file then becomes a no-op with no warning.
- A sibling AGENTS.md must exist in the same directory.
- Add one of these shims in each directory that has a nested AGENTS.md.
  Claude loads a nested CLAUDE.md when it reads files under that directory.
- Only Claude-specific behavior may go below this comment. Example: "Use
  plan mode for src/billing/." A rule that applies to all agents goes in
  AGENTS.md.
- Claude strips HTML comments before injection. This note costs no context.

Verify after each edit: start `claude` in this directory. Run `/context`.
Confirm that Memory files lists CLAUDE.md with the imported content.
-->
