# Prompt 15 — Record the NonProduction confirmation authority

Using the reusable contract and the artifacts produced by Prompts 1–14, including 12F, record who
supplies the posting confirmation response in this home lab, for `NonProduction` deployments only.

This is a documentation stage. It changes no code path, no configuration setting, and no test.
Prompt 5 already built the confirmation as an opaque message authentication code the server mints
and verifies; the server has no way to observe, and no code change can give it a way to observe,
*who* rendered the plan summary and returned the answer. That is a deployment property outside this
server's reach, exactly as `docs/threat-model.md`'s prompt-injection section already says. This
prompt states one reviewed answer to that deployment property for this home lab. It does not, and
cannot, make the server verify it.

## The decision

`docs/supplier-invoice-posting.md` currently leaves the trusted client open between two shapes: "an
MCP client's own human-approval interaction, or an approval gateway in front of this server." For
every `NonProduction` deployment on this box — `SapUpstreamClass.Fake` and `SapUpstreamClass.Sandbox`
alike, not only a demo preset — the repo owner has reviewed and chosen the first: **the calling MCP
client itself, driven interactively by the human in that conversation, is the trusted client.** No
separate approval-gateway service sits between `sap_plan_supplier_invoice` and
`sap_post_supplier_invoice` in this topology, and none is required by it.

State plainly what this is and is not:

- It narrows an open choice Prompt 5 left to the deployment; it does not reverse Prompt 5's rule
  that a model's own prose is not proof, and it does not add a `confirmed: true` shortcut. The
  challenge is still a plan-bound MAC the server mints and verifies, and the human in the
  conversation is still the one who has to see the rendered summary and approve it before the
  client returns the confirmation response. What changes is only which actor plays "the trusted
  client" — the conversational agent's own interaction with its human, not a distinct gateway
  process this server or LocalAI would otherwise have to stand up and operate.
- It is scoped to `NonProduction` only. This prompt takes no position on what a `Production`
  deployment's trusted client must be — that remains an open deployment decision for whoever
  operates one, and the existing open wording in `docs/supplier-invoice-posting.md` and
  `docs/threat-model.md` continues to apply there. Do not narrow the `Production` case by
  implication while writing the `NonProduction` one.
- It changes nothing about the confirmation mechanism itself: the message authentication code, the
  plan's binding to caller, tenant, company code, normalized-invoice fingerprint, configuration
  fingerprint and expiry, the atomic single-use consumption, the 15-minute plan lifetime, and the
  zero-retry posting rule are all exactly as Prompt 5 and Prompt 10 left them. Nothing here touches
  authorization, company-code or amount boundaries, or the plan store.
- It is a reviewed **acceptance** of a residual risk `docs/threat-model.md`'s prompt-injection
  section already names, not a new mitigation: "a deployment that lets the model round-trip its own
  challenge has a server that both asks and answers, which is not a confirmation." The calling
  agent surfacing the plan summary to the human it is working with, and relaying the human's
  answer, is the "MCP client's own human-approval interaction" that section already treats as a
  legitimate trusted-client shape — this prompt names it as the one this home lab actually uses,
  it does not introduce the model answering its own challenge unattended.

## Do not add a code gate

There is no `SapDeploymentProfile`, `SapUpstreamClass`, feature flag, configuration setting, or
runtime check to add for this decision, and none should be added. The server cannot distinguish "the
calling agent, under human direction" from any other value on the wire — the confirmation response
is the same opaque string either way, and a check that pretended otherwise would be decoration, not
a control. If a later stage needs the server to actually verify a distinct approval-gateway
interaction, that is new mechanism, not this prompt, and it is not scoped here.

## Update the documentation

- `docs/supplier-invoice-posting.md` — in "The confirmation is not a Boolean," state the
  `NonProduction` decision immediately after the existing two-shape sentence, labelled explicitly as
  a reviewed home-lab decision and distinguished from the still-open `Production` question. Do not
  delete or reword the existing sentence; add to it.
- `docs/threat-model.md` — in the prompt-injection section's residual-risk paragraph, add the same
  statement: for `NonProduction`, this deployment's answer to "which trusted client" is the calling
  agent under human direction, reviewed and accepted, and the round-trip failure mode the paragraph
  warns about is what is being deliberately avoided by keeping a human between the plan summary and
  the confirmation — not what this decision permits.
- Any deployment-facing document that currently repeats the same two-shape open question (check
  `docs/deployment-audit.md` and `docs/operations.md` for a restatement of it) gets the identical
  addition, worded identically, so a reader of any one of them gets the same answer. Do not restate
  the decision in different words in different files — link or quote the canonical statement in
  `docs/supplier-invoice-posting.md` if a second file needs to reference it.

## Prove

There is no executable test for a documentation-only decision about an actor outside the process.
Prove instead, by inspection, that:

- every place in the repository's documentation that poses the "MCP client's own human-approval
  interaction, or an approval gateway" choice as open now also states the `NonProduction` answer, in
  the same words, next to it;
- no file states or implies the answer applies to `Production`;
- no source file under `src/` changed, and no test under `test/` changed or was added — a diff
  outside `docs/` and `Narrative.md`/`narrative/` is a defect in this stage, not evidence of it;
- `docs/supplier-invoice-posting.md`'s existing description of the MAC, the plan bindings, the
  15-minute expiry, single-use consumption, and the zero-retry post rule is byte-identical to before
  this stage, aside from the addition above; and
- the full existing test suite still passes unchanged, which is the mechanical proof that nothing
  behavioural moved.

## Acceptance criteria

- The `NonProduction` confirmation-authority decision is recorded once, in `docs/supplier-invoice-
  posting.md`, and referenced rather than duplicated everywhere else it is relevant.
- The decision is explicitly scoped to `NonProduction` and explicitly silent on `Production`.
- No code, configuration, or test changed.
- The plan/confirmation mechanism — MAC, bindings, expiry, single-use consumption, zero-retry
  posting — is unchanged and undocumented as anything other than unchanged.
- Formatting and the full existing test suite succeed, proving no behavioural drift.

Commit locally. Use `narrative-required` and record the decision, why it is scoped to `NonProduction`
only, why no code gate was added and why one would be decoration rather than a control, and that it
narrows an open choice Prompt 5 already established rather than reversing any rule. Do not push
unless requested.
