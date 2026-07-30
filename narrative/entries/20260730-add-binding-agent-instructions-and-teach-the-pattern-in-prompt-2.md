---
date: 2026-07-30
slug: add-binding-agent-instructions-and-teach-the-pattern-in-prompt-2
title: "Add binding agent instructions, and teach the pattern in Prompt 2"
summary: "`CLAUDE.md` is this repository's single source of truth, with six content-free pointers. §1 records the prompt-sequence constraints; §2 the narrative contract; §3 git discipline including the no-stacking rule."
kind: product
status: accepted
sequence: 2026-07-30T05:29:42.000Z
evidence: "https://github.com/jamiemitchellconsultants/SAPProxyMCPServerBuilder/pull/4; merge commit 9b720a2c60698997ab6c832805027ba7423a61ca"
---

## Context

Two gaps, one of which propagates.

This repository had no agent instruction files. Nothing told an agent that a prompt is a specification handed to an agent in a *different* repository, that the reusable contract is referenced by every later stage, that stages are submitted in order, or that acceptance checks are never weakened to make a stage pass.

More consequentially, **Prompt 2 installed the narrative mechanism without requiring anything that would tell a later agent it existed.** It scaffolded the workflows, the template and the follow-ups, then moved on. Stages 3 through 10 are the decision-bearing ones, and they would each be executed by an agent with no knowledge of the label rule.

That is not hypothetical. A sibling repository built from an equivalent prompt sequence lost entries on four consecutive pull requests. Its template documented the label rule correctly throughout; the agent never read it, because `gh pr create --body ...` replaces the repository template wholesale. Three entries had to be reconstructed by hand, which the merge-event-only trigger makes the only repair.

A second failure in the same repository: the `narrative-required` label was applied before the body carried the three sections, and the pull request merged inside that window. The maintenance run failed permanently, because the action reads the body from the merge event payload — a re-run reads the same incomplete text. `OntologyService` already prevents exactly this with a pre-merge `require-narrative-sections` job; this sequence taught nothing equivalent.

## Decision

`CLAUDE.md` is this repository's single source of truth, with six content-free pointers. §1 records the prompt-sequence constraints; §2 the narrative contract; §3 git discipline including the no-stacking rule.

Prompt 2 gains two requirements for the built server, both stated as consequences of observed failures rather than as style:

- **Agent instruction files.** `CLAUDE.md` as the single source of truth covering the label-and-sections rule applied in one action, the merge-event-only limitation, the supplied-body caveat, the never-hand-merge rule for the compiled narrative, and the never-rewrite-an-accepted-entry rule — plus six thin pointers that restate none of it, because a stale copy is worse than no copy.
- **A pre-merge narrative gate.** A second job in the validation workflow that fails a `narrative-required` pull request whose body lacks any of the three non-empty sections, triggered on `labeled` and `unlabeled` so it re-evaluates when classification changes. It must read the body from an environment variable, since pull-request prose is untrusted input.

The gate deliberately does not check for a *missing* label. Only a human can classify a change; the job catches the case where classification was declared and the evidence was not supplied.

Prompt 2 rather than a later stage, because the mechanism has to be discoverable from the first decision-bearing stage onward, not retrofitted at delivery.

## Consequences

Servers built from this sequence now arrive with the narrative contract discoverable by any tier-one agent and with the label-without-sections failure caught pre-merge instead of permanently.

Prompt 2's scope grows, and its acceptance criteria gain four checks. It remains an unlabelled mechanical installation stage — the workflow still cannot capture the pull request that first installs it.

The pointer set is a maintenance surface in every built repository: a new tool means a new file and a list update in the others.

Not addressed: an unlabelled decision-bearing pull request still produces silence, in this repository and in anything built from it. Catching that needs a check that judges whether a change is decision-bearing, which is a human call.
