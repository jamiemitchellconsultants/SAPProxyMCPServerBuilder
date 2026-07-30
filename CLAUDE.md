# SAPProxyMCPServerBuilder — Claude Code Instructions

> Read this file in full before changing anything. Every rule here is binding — regardless of which
> AI tool is reading it. This file is the single source of truth; `AGENTS.md`, `GEMINI.md`,
> `.github/copilot-instructions.md`, `.cursor/rules/`, `.windsurf/rules/`, and `.clinerules/` are
> thin pointers back to it (for Codex/OpenCode, Gemini CLI, GitHub Copilot, Cursor, Windsurf, and
> Cline respectively) so the rules never have to be kept in sync across multiple copies. Edit this
> file, not those.

Read [README.md](README.md), [START-HERE.md](START-HERE.md) if present, and the **Index** at the top
of [Narrative.md](Narrative.md) before your first change.

---

## §1 — What this repository is

A staged prompt sequence that teaches a coding agent to build a narrowly scoped SAP supplier-invoice
MCP server. This repo contains `prompts/`, **not the server** — the implementation lives in
`SAPProxyMCPServer`.

Consequences for how you work here:

- A prompt is a specification handed to an agent working in a *different* repository. It cannot
  assume anything exists that an earlier prompt did not create.
- Prompts are submitted in order, one per agent task, each only after the previous stage passed its
  acceptance checks. A change that reorders, renumbers, or splits a stage must keep that true and
  update `README.md`'s sequence listing in the same change.
- `prompts/00-reusable-contract.md` is submitted first and referenced by every later stage. Changing
  it changes every stage; say so explicitly when you do.
- Never weaken an acceptance check to make a stage easier to pass. The sequence's value is that each
  boundary is real.
- The taught server's capability surface is deliberately closed. A prompt that lets an agent widen
  it — an extra operation, field, or status value — is a defect, not a feature.

---

## §2 — Narrative discipline

This repository treats the history of decisions as a primary artifact. Every session that makes a
meaningful decision — product, architecture, governance, operational, a correction to an earlier
decision, or an experiment worth remembering — records an entry. This is not optional documentation.

### Binding rule — you write fragments, never `Narrative.md`

`Narrative.md` is **generated output**. Its own first line says so. **Never author it, hand-edit it,
or hand-merge it.** The only narrative file you ever write by hand is a fragment under
`narrative/entries/`.

Three cases get this wrong in different ways:

1. **Adding an entry** — write a fragment. Never append an entry body or an index row to
   `Narrative.md`. The index is derived; a hand-added row is destroyed by the next compile.
2. **Changing existing wording** — edit that entry's fragment, never the compiled projection.
3. **Resolving a merge conflict in `Narrative.md`** — never resolve it by hand, however trivial the
   markers look. Discard both sides and regenerate. Fragments are the source of truth and merge
   cleanly, because each entry is a separate file; only the projection collides. Two branches that
   each added an entry produce a conflict whose correct resolution is the union of the fragments,
   which the compiler computes and you should not.

Running the compiler is **not** authoring the file. Compilation is deterministic and model-free, so
the output is a function of the fragments and nothing else. Compile it; never type it.

```bash
npx --yes --package=github:jamiemitchellconsultants/Narrative narrative compile
```

The compiled file **is** committed alongside the fragment that changed, because
`validate-narrative.yml` runs `check`, which fails when the output is stale. Committing generated
output is not the same as authoring it.

### The label and the pull-request body are both required

Entries are produced by the [Project Narrative][pn] action. On merge it reads the pull request and
proposes a fragment on a separate draft pull request. It needs **two** things:

1. The `narrative-required` label.
2. Three non-empty headings in the pull-request **body**, named exactly:
   - `## Narrative Context`
   - `## Narrative Decision`
   - `## Narrative Consequences`

**Apply both in the same action — never the label first.** A labelled pull request whose body lacks
the sections fails the maintenance run; if it merges in that window the failure is permanent,
because the action reads the body from the merge event payload and a re-run reads the same
incomplete text.

Missing label: the workflow exits quietly and no entry is ever produced. Missing sections with the
label present: the workflow **fails visibly**. Neither is recoverable after merge — the workflow
triggers on the merge event only, so labelling a merged pull request does nothing. A missed entry
has to be written by hand as a fragment.

`.github/pull_request_template.md` carries these sections. **Do not bypass the template.** Creating
a pull request with `gh pr create --body ...` replaces it wholesale, which is the single easiest way
to lose an entry. If you pass `--body`, carry the three sections in it yourself.

Do **not** apply `narrative-required` to a narrative-only pull request (a proposed fragment, or a
repair), because that would recursively create an entry about maintaining the narrative.

### Writing a fragment by hand

Only when an entry was missed, or when correcting the record. Create
`narrative/entries/YYYYMMDD-<slug>.md`:

- Required front matter: `date` (`YYYY-MM-DD`), `slug` (lower-case kebab-case; the filename after
  the date prefix must match it exactly), `title`, `summary` (within `summaryMaxCharacters` from
  `.project-narrative.json`), `kind`, `status`.
- Optional: `sequence`, a full-precision UTC instant ordering same-day entries in true merge order;
  and `evidence`, the pull-request URL plus merge commit.
- `kind` is one of `architecture`, `product`, `governance`, `operational`, `correction`,
  `experiment`. `status` is one of `proposed`, `accepted`, `superseded`.
- Body: exactly `## Context`, `## Decision`, `## Consequences`, in that order, none empty. Note the
  plural on the last; the validator rejects `## Consequence`.

### Never rewrite an accepted entry

An accepted entry records what was decided and why **at the time**. When a later decision refines or
reverses it, add a new entry with `kind: correction` linking back by slug — do not edit the original
so it reads as though the better framing had been there all along. That destroys the evidence that
the framing ever needed correcting.

Cite entries by slug (`#entry-<slug>`), never by number. Numbers are positional and shift.

---

## §3 — Git and review discipline

- Branch names follow `category/short-name`.
- **Do not stack a pull request on another pull request's branch.** If the base merges first the
  stacked branch is orphaned: GitHub reports it as merged while its commits never reach the default
  branch. Branch from the default branch and accept a textual reference to an unmerged decision.
- After pushing follow-up commits to a branch with an open pull request, say so explicitly. A pull
  request merged before later commits arrive silently drops them, and the merge looks clean.
- Verify what actually landed with `git log origin/main --oneline` after a merge, rather than
  trusting the pull request's reported state.
- Preserve unrelated working-tree changes. Commit or push only when asked, and never force-push a
  shared branch or delete a remote branch unless explicitly requested.

## §4 — Prose conventions

- Markdown wraps at 100 columns.
- No emoji.
- Absolute dates (`YYYY-MM-DD`), never relative ones.
- State limitations plainly rather than qualifying them away. Do not report a skipped or
  unverifiable check as passing; label it a manual gate with the exact command and expected result.

[pn]: https://github.com/jamiemitchellconsultants/Narrative
