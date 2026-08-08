---
date: 2026-08-08
slug: add-prompt-15-record-the-nonproduction-confirmation-authority
title: "Add Prompt 15 — record the NonProduction confirmation authority"
summary: "For every `NonProduction` SAP deployment on the home-lab box (`SapUpstreamClass.Fake` and `SapUpstreamClass.Sandbox` alike, not only a demo preset), the calling MCP client itself — driven interactively by the human in that conversation…"
kind: product
status: accepted
sequence: 2026-08-08T04:07:14.000Z
evidence: "https://github.com/jamiemitchellconsultants/SAPProxyMCPServerBuilder/pull/8; merge commit f535947d5e446c80e6b517332608982a315c481f"
---

## Context

Prompt 5 built `sap_post_supplier_invoice`'s confirmation as an opaque message authentication code the server mints and verifies, but deliberately left open *who* renders the plan summary to a person and returns the answer — `docs/supplier-invoice-posting.md` states the choice as "an MCP client's own human-approval interaction, or an approval gateway in front of this server," and never resolves it. LocalAI's `plan_brightflag_sap_demo.md` needed that question answered before deploying the SAP MCP server for the BrightFlag→SAP demo workflow, since no separate approval-gateway service exists in that home-lab topology.

## Decision

For every `NonProduction` SAP deployment on the home-lab box (`SapUpstreamClass.Fake` and `SapUpstreamClass.Sandbox` alike, not only a demo preset), the calling MCP client itself — driven interactively by the human in that conversation — is the trusted client that renders the plan summary and returns the confirmation. No separate approval-gateway service exists or is required in this topology. This is documentation only: the server cannot observe who supplied a confirmation response (it is the same opaque string either way), so no code gate was added, and none should be — one would be decoration, not a control. The confirmation MAC, plan bindings, 15-minute expiry, single-use consumption, and zero-retry posting rule are all unchanged. The `Production` case is left exactly as open as it already was; this prompt does not narrow it by implication.

## Consequences

Once this prompt is played into `SAPProxyMCPServer`, its `docs/supplier-invoice-posting.md` and `docs/threat-model.md` will state the `NonProduction` confirmation authority explicitly rather than leaving it as an open deployment choice, which is what let LocalAI deploy the SAP MCP server for the demo workflow without inventing or standing up a separate approval-gateway process. A future `Production` deployment still has to make and document its own choice between the two shapes Prompt 5 left open — this decision does not answer that for it, and a later prompt would be needed if the server should ever be made to verify the distinction rather than merely document it.
