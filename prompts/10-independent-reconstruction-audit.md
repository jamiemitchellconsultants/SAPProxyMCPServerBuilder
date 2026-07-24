# Prompt 10 — Independent reconstruction audit

Use this prompt as a fresh task or, ideally, with a different coding agent.

Audit the completed SAP supplier-invoice MCP server against the reusable contract and all staged
prompts. Do not add new capabilities. Fix only confirmed defects.

## Verify the capability ceiling

Confirm independently:

1. The MCP server advertises exactly four tools and one ontology resource.
2. Runtime configuration contains exactly the two fixed SAP OData V2 service roots.
3. No caller can supply an origin, service, path, entity, action, method, header, raw filter, query
   string, role, or credential.
4. Live metadata cannot create a tool, field, invoice shape, or operation at runtime.
5. Version 1 accepts only the closed purchase-order-referenced invoice schema.
6. Release, reverse, delete, park, hold, batch, G/L-only, asset, material, one-time-supplier, and
   arbitrary action behavior is absent.

## Verify invoice posting

7. Planning validates caller, tenant, company code, amount, dates, currency, tax policy, PO
   references, and closed JSON shape without contacting SAP.
8. Plans are cryptographically random, expiring, single-use, race-safe, and bound to caller, tenant,
   normalized payload, company code, configuration, and policy.
9. Posting requires trusted confirmation and cannot be triggered by a Boolean, tool annotation,
   prompt text, or model claim.
10. Authorization denial happens before credential or SAP access.
11. CSRF token acquisition and the deep-create POST share the fixed destination and scoped session.
12. The only mutation is one POST to
    `API_SUPPLIERINVOICE_PROCESS_SRV/A_SupplierInvoice`.
13. The SAP response is validated and exposes only supplier invoice, fiscal year, approved status,
    correlation, and audit identifiers.
14. Invoice posting is never retried, including timeout and connection-loss paths.
15. An ambiguous outcome is reported as indeterminate with reconciliation guidance.

## Verify recently paid evidence

16. The read path uses only `API_OPLACCTGDOCITEMCUBE_SRV`.
17. Lookback defaults to 7 days, remains between 1 and 31, and results never exceed 100.
18. “Paid” requires a supplier payable item, `IsCleared`, clearing date, clearing document, and
    authoritative invoice/accounting linkage from reviewed metadata.
19. The server verifies that no payable item for a candidate invoice remains open.
20. Partial, open, ambiguous, invalidated, non-supplier, and out-of-window records are excluded.
21. Queries use server-generated bounded filters, projection, ordering, pages, bytes, timeout, and
    concurrency.
22. Returned clearing evidence is deterministic, redacted, and deduplicated by stable invoice
    identity.

## Verify ontology reporting

23. Tool input/output JSON Schemas match runtime contracts.
24. Ontology entities and relationships cover supplier invoice posting and clearing evidence without
    inventing facts.
25. Source provenance names only the two fixed SAP services.
26. Schema compilation is byte-identical and stale check never writes.
27. Tool and resource return the same fingerprinted checked-in schema without SAP access.
28. Schema artifacts contain no invoice instances, payment rows, supplier data, credentials,
    hostnames, or environment-specific values.

## Verify platform controls

29. Stdio is fake-only and remote HTTP is authenticated stateless Streamable HTTP.
30. Bearer validation checks signature, issuer, audience, lifetime, and required claims.
31. Permissions, tenant, company code, and amount boundaries are deny-by-default.
32. Rate, concurrency, message, response, cancellation, and shutdown behavior is bounded.
33. Logs, metrics, traces, errors, audits, snapshots, fixtures, and generated artifacts contain no
    secrets or sensitive business data.
34. Automated tests contact only the local fake SAP server.
35. Container, CI, documentation, threat model, and Narrative governance match the implementation.

## Adversarial tests

Exercise:

- URL, path, OData, header, XML, JSON, decimal, and log injection;
- unknown JSON fields and attempts to submit non-PO invoice nodes;
- CSRF or cookie theft and cross-destination reuse;
- plan replay, expiry, argument swapping, cross-caller use, races, and policy change;
- a mutation timeout after the fake SAP server receives the request;
- attempts to invoke release, reverse, delete, park, hold, batch, and arbitrary actions;
- partial clearing presented as paid;
- clearing date boundary and timezone manipulation;
- candidate invoices with residual open items;
- ambiguous or reversed clearing evidence;
- oversized pages, errors, and continuation data;
- cross-company and cross-tenant reads;
- schema additions for a fifth tool or third SAP service; and
- seeded secrets and business identifiers at every telemetry and generation boundary.

## Commands

From a clean clone, run at minimum:

```bash
dotnet restore --locked-mode
dotnet format --verify-no-changes
dotnet build --no-restore
dotnet test --no-build
dotnet list package --vulnerable --include-transitive
dotnet run --project src/SapInvoiceMcpServer -- schema check
npx --yes --package=github:jamiemitchellconsultants/Narrative narrative check
git diff --check
docker build .
```

If Docker is unavailable, report the gap. Do not contact live SAP to compensate for missing
fake-server evidence.

## Report

Report findings by severity with exact file and line references. For each confirmed defect, explain
the violated requirement, implement the smallest correction, add a regression test, and rerun the
affected and complete checks.

Finish with a requirements-to-evidence table for all 35 verification points. List residual risks,
unrun checks, manual SAP controls, and any target-release metadata incompatibility.

Commit audit fixes locally if needed. Do not push, label, open, or merge a pull request unless
explicitly requested.
