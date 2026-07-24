# Prompt 3 — Configure the two fixed SAP services

Using the reusable contract, implement Stage 3: secret-free configuration and reviewed metadata for
exactly two SAP OData V2 services.

Configure:

- one SAP origin per deployment, outside MCP arguments;
- optional three-digit SAP client;
- authentication and credential-provider references;
- company codes the caller may use;
- request timeout, concurrency, response-byte, and page limits;
- supplier-invoice metadata snapshot;
- operational-accounting metadata snapshot; and
- the two fixed relative service roots:
  - `/sap/opu/odata/sap/API_SUPPLIERINVOICE_PROCESS_SRV`
  - `/sap/opu/odata/sap/API_OPLACCTGDOCITEMCUBE_SRV`

Reject:

- any third service;
- caller-controlled origins or paths;
- URL user information, fragments, wildcards, traversal, or query strings;
- plain HTTP except for the loopback fake server;
- redirects to another origin;
- credentials embedded in JSON or URLs;
- production profiles using no authentication;
- unapproved company codes; and
- unknown configuration properties.

Support secret-provider contracts for OAuth client credentials and client certificates, plus a fake
provider for tests. Basic authentication may be represented only when required by a supported
on-premise sandbox and must be rejected for production-by-default examples. Do not implement a
vendor-specific vault.

## Metadata snapshots

Add an explicit administrative command that fetches `$metadata` only for the two configured roots.
It must use bounded HTTPS, disable DTDs and external XML resolution, reject cross-origin redirects,
write atomically, and print a review diff.

Validate from reviewed metadata that:

- `API_SUPPLIERINVOICE_PROCESS_SRV` contains `A_SupplierInvoice`, the PO-reference navigation
  required for deep create, and the response keys;
- `API_OPLACCTGDOCITEMCUBE_SRV` contains the operational item entity plus the exact fields required
  for supplier, company code, fiscal year, accounting document, clearing status, clearing date,
  clearing document, currency, and amount.

If required fields are absent for the target SAP release, fail configuration. Do not substitute a
similar field or silently reduce the evidence contract.

Runtime startup reads checked-in snapshots only. It never fetches metadata or grows the MCP surface.

## Fake SAP server

Implement a loopback fake server exposing only:

- `$metadata` for the two services;
- `POST .../API_SUPPLIERINVOICE_PROCESS_SRV/A_SupplierInvoice`;
- CSRF token acquisition for that service; and
- bounded `GET` on the operational-accounting item entity.

Use synthetic invoices, suppliers, purchase orders, payments, and errors.

## Acceptance criteria

- Configuration resolves exactly two fixed service roots.
- Unsafe URLs, third services, secrets, unknown properties, and unapproved company codes fail.
- Metadata validation proves every required posting and clearing field.
- Runtime startup is metadata-network-free.
- Fake-server tests assert exact paths and reject every other route.
- Formatting, build, and tests succeed.

Commit locally. Use `narrative-required` on the implementation pull request and record why the
server uses two fixed services instead of a general SAP registry. Do not push unless requested.
