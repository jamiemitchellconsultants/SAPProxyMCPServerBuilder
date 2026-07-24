# Prompt 9 — Package, document, and govern the server

Using the reusable contract, implement Stage 9: production packaging, CI, operating documentation,
and audit guidance for the narrow supplier-invoice server.

## Delivery

Add:

- a multi-stage .NET 10 container build;
- locked deterministic restore;
- a deliberately pinned supported runtime image;
- an unprivileged runtime user;
- a read-only filesystem where deployment permits;
- liveness that never contacts SAP;
- readiness based on local configuration and schema state, not SAP availability; and
- graceful termination that stops new invoice posts before draining work.

The runtime image must not contain SDK tooling, test fixtures, metadata-pull tooling, credentials,
or example invoice bodies unless operationally required and documented.

Add least-privilege GitHub Actions for:

- locked restore;
- format verification;
- warnings-as-errors build;
- unit and fake-SAP integration tests;
- ontology schema stale check;
- Narrative validation;
- dependency vulnerability reporting;
- secret scanning where available; and
- container build without publishing.

Normal CI must never contact SAP. A live sandbox test remains a separately authorized manual
workflow, read-only unless an SAP administrator deliberately provisions synthetic invoice posting.

## Documentation

Complete:

- `README.md`
- `docs/architecture.md`
- `docs/configuration.md`
- `docs/supplier-invoice-posting.md`
- `docs/recently-paid-invoices.md`
- `docs/ontology-registration.md`
- `docs/authentication-authorization.md`
- `docs/operations.md`
- `docs/threat-model.md`
- `SECURITY.md`
- `CONTRIBUTING.md`

Document:

- the exact four tools and one resource;
- the two fixed service roots;
- the PO-backed version-1 invoice schema and exclusions;
- SAP communication arrangements and least-privilege identities;
- plan, trusted confirmation, CSRF, posting, response, and reconciliation flow;
- why posting is never automatically retried;
- the evidence-based definition of recently paid;
- lookback, full-clearing verification, paging, and volume limits;
- ontology entities, relationships, provenance, fingerprint, and registration;
- stdio fake-only and remote authenticated deployment;
- credential rotation and incident response;
- audit-event ownership and retention;
- schema/config upgrade and rollback;
- synthetic fake-SAP walkthrough; and
- known limitations.

Examples must use synthetic placeholders. Never include a real host, company code, supplier,
purchase order, invoice, clearing document, amount, credential, token, certificate, or error.

## Governance

Add canonical coding-agent instructions preserving:

- four tools and one resource only;
- two fixed SAP services only;
- no arbitrary OData;
- PO-backed invoice posting only;
- plan-before-post and no POST retry;
- full-clearing evidence for recently paid;
- deterministic data-free ontology schema;
- fake-SAP automated tests;
- external secrets and redacted telemetry; and
- Project Narrative's two-pull-request lifecycle.

Verify Narrative without reinstalling it. Meaningful changes to the invoice shape, paid definition,
lookback, SAP services, tool surface, authorization, or ontology contract require
`narrative-required` and exact Context, Decision, and Consequences.

## Acceptance criteria

- A clean clone follows documented restore, build, test, schema, and container commands.
- CI contacts only the fake SAP server.
- Documentation matches executable tools and schemas exactly.
- Threat model covers prompt injection, confused deputy behavior, SSRF, metadata poisoning,
  credential leakage, over-broad SAP authority, invoice replay, ambiguous posting, false payment
  classification, cross-tenant access, and schema poisoning.
- Complete checks and container build succeed when Docker is available.

Commit locally. Use `narrative-required` and record the delivery and governance decision. Do not
push unless requested.
