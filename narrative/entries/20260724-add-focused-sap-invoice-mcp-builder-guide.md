---
date: 2026-07-24
slug: add-focused-sap-invoice-mcp-builder-guide
title: "Add focused SAP invoice MCP builder guide"
summary: "Replace the broad proxy curriculum with a capability-limited sequence."
kind: product
status: accepted
evidence: "https://github.com/jamiemitchellconsultants/SAPProxyMCPServerBuilder/pull/1; merge commit 835099abf32299ba7a98b6cbd05029c2e78fec1a"
---

## Context

The initial builder guide described a broad governed SAP proxy. The required product scope was subsequently narrowed to three capabilities: post a supplier invoice through `API_SUPPLIERINVOICE_PROCESS_SRV`, identify recently paid invoices, and report a schema suitable for ontology-server registration.

## Decision

Replace the broad proxy curriculum with a capability-limited sequence. Expose only invoice planning/posting, recently paid invoice reporting backed by clearing evidence from `API_OPLACCTGDOCITEMCUBE_SRV`, and deterministic ontology schema reporting. Keep arbitrary SAP and OData execution outside the server contract.

## Consequences

The guide now teaches a smaller and more auditable MCP surface with two fixed SAP services and four tools. Invoice posting uses a closed PO-backed version-one shape with plan-before-execute confirmation and no automatic POST retry. Payment reporting requires full-clearing evidence rather than invoice status. Supporting another invoice shape, SAP service, or tool requires a separately reviewed contract change.
