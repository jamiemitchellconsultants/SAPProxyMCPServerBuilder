# Prompt 2 — Adopt Project Narrative

Using the reusable contract, implement Stage 2: install Project Narrative before decisions about
invoice posting, payment evidence, and ontology reporting are implemented.

Project Narrative is a deterministic, review-first decision-history mechanism. It is not a changelog
generator and must never invent rationale from code or diffs.

## Read and run the current installer

Read the current `install.md` in `jamiemitchellconsultants/Narrative` before editing. From the root
of the consumer repository, run the installer exactly as its current contract specifies:

```bash
npx --yes --package=github:jamiemitchellconsultants/Narrative narrative install
```

Do not reconstruct generated files from memory and do not overwrite an existing pull-request
template. If a template already exists, add the three required headings without changing their
spelling:

- `## Narrative Context`
- `## Narrative Decision`
- `## Narrative Consequences`

Set the Narrative title to `SAP Proxy MCP Server Narrative` only through supported configuration. Do
not hand-edit `Narrative.md`; fragments are authoritative and the compiled file is a projection.

Run:

```bash
npx --yes --package=github:jamiemitchellconsultants/Narrative narrative check
```

Report the result honestly. Do not weaken validation.

## Bootstrap and repository actions

This installation is mechanical and must not carry the `narrative-required` label. The workflow
cannot capture the pull request that first installs it because workflows must already exist on the
default branch.

Surface the installer's manual follow-ups exactly:

1. Enable read/write Actions workflow permissions and allow GitHub Actions to create and approve
   pull requests.
2. Create the repository label named exactly `narrative-required`.
3. Commit and merge the scaffolded files to the default branch before Prompt 3.

Do not report any repository setting or label as completed unless you actually verified or changed
it through an authorized repository action.

Document the two-pull-request lifecycle and make clear that later changes to the fixed service
paths, invoice payload, definition of paid, lookback limit, exposed MCP tools, or ontology contract
are decision-bearing:

1. A meaningful project pull request carries `narrative-required` and explicit Context, Decision,
   and Consequences.
2. After it merges, automation opens a separate narrative proposal.
3. Review and merge that proposal without `narrative-required` to avoid recursion.

## Agent instruction files

The narrative lifecycle above only works if whichever coding agent runs a later stage actually knows
about it. Do not leave that to the agent's defaults.

Write `CLAUDE.md` at the repository root as the **single source of truth** for agent instructions,
binding regardless of which tool reads it. It must cover, at minimum: that `Narrative.md` is
generated and is never authored, hand-edited, or hand-merged, the only hand-written narrative file
being a fragment; that a decision-bearing pull request needs both the `narrative-required` label and
three non-empty body headings, applied **in the same action** rather than label-first; that the
maintenance workflow fires on the merge event only, so neither omission is repairable afterwards and
a missed entry has to be hand-written as a fragment; that a narrative-only pull request carries no
label; and that an accepted entry is never rewritten — a reversal is a new entry of kind
`correction` citing the original by slug.

State plainly that creating a pull request with a supplied body replaces the repository template
wholesale, and that doing so without carrying the three sections forward is the most common way an
entry is silently lost.

Record the merge-conflict rule too: when two branches each add an entry, the fragments merge cleanly
because each entry is a separate file, and only the compiled projection collides — so discard both
sides of the projection and recompile rather than reconciling the markers.

Then add **thin pointer files** for the other tier-one agents, each saying only that `CLAUDE.md` is
authoritative, that its rules are binding, and that instruction changes go in `CLAUDE.md`:

- `AGENTS.md` — Codex, OpenCode, and any agent reading the generic file;
- `.github/copilot-instructions.md` — GitHub Copilot;
- `GEMINI.md` — Gemini CLI;
- `.cursor/rules/agent-instructions.mdc` — Cursor, with `alwaysApply: true` front matter;
- `.windsurf/rules/agent-instructions.md` — Windsurf, with `trigger: always_on` front matter;
- `.clinerules/agent-instructions.md` — Cline.

The pointers must **not** restate the rules. Duplicated instructions drift, and a stale copy is
worse than no copy because an agent cannot tell which is current. An agent whose tool has no pointer
sees no instructions at all, so a missing pointer is not cosmetic. Do not list a pointer location
the repository does not actually contain.

This file set is mechanical scaffolding and belongs in the unlabelled installation change.

## Pre-merge narrative gate

The scaffolded validation workflow checks fragment validity and compiled freshness. Add a second job
to it, triggered so that it re-evaluates whenever the label or the body changes:

```yaml
on:
  pull_request:
    types: [opened, edited, reopened, synchronize, labeled, unlabeled]
```

**When the pull request carries `narrative-required`**, the job fails unless the body contains all
three non-empty `## Narrative …` sections.

This mirrors the post-merge gate before merge, and it exists because of a specific, observed
failure: the label can be applied while the body still lacks the sections, and if the pull request
merges in that window the maintenance run fails permanently. The action reads the body from the
merge event payload, so re-running it reads the same incomplete text and the entry must be
hand-written instead. A pre-merge check turns that into a red result on an open pull request, which
is fixable.

The job must read the body from an environment variable rather than interpolating it into a shell
command — pull-request prose is untrusted input.

It deliberately does not check for a *missing* label; only a human can classify a change. It catches
the case where classification was declared and the evidence was not supplied.

## Acceptance criteria

- Configuration, preamble, generated narrative, workflows, and pull-request template exist.
- `narrative check` returns zero.
- Workflows and template use the exact same label and section headings.
- No narrative rationale was invented.
- `CLAUDE.md` exists and states the label rule, the three required body headings, the
  merge-event-only limitation, the supplied-body caveat, and the never-hand-merge rule.
- Every pointer file exists, names `CLAUDE.md` as authoritative, restates none of its rules, and
  references no location the repository does not contain.
- The validation workflow fails a `narrative-required` pull request whose body is missing any of the
  three sections, and reads the body from an environment variable rather than a shell interpolation.
- The installation pull request is unlabelled.
- The learner is explicitly told to stop until the installation is published and merged.

Commit the bootstrap locally with a focused message. Do not push automatically. Pause the learning
sequence until the learner explicitly publishes and merges the installation.
