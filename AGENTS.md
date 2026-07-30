# SAPProxyMCPServerBuilder — Agent Instructions

This repository's binding AI agent instructions live in [`CLAUDE.md`](CLAUDE.md) at the repo root.
Read it in full before generating any code or proposing any change — every rule there is binding,
regardless of which AI tool is being used.

`CLAUDE.md` is the single source of truth. This file, and the equivalent files for other tools
(`GEMINI.md`, `.github/copilot-instructions.md`, `.cursor/rules/`, `.windsurf/rules/`,
`.clinerules/`), are thin pointers to it — kept that way deliberately so the rules never have to be
kept in sync across multiple copies. If you are updating the instructions, edit `CLAUDE.md`, not
this file.

Note in particular the Project Narrative rules. A decision-bearing pull request needs both the
`narrative-required` label and these non-empty sections in its body:

- `## Narrative Context`
- `## Narrative Decision`
- `## Narrative Consequences`

Neither omission can be repaired after merge, so apply both before it. `CLAUDE.md` also records that
generated narrative output is never authored, hand-edited, or hand-merged: the only narrative file
written by hand is a fragment.
