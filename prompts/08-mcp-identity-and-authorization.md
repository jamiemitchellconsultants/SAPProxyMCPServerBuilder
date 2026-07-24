# Prompt 8 — Expose the narrow MCP surface securely

Using the reusable contract, implement Stage 8: the four fixed MCP tools and one schema resource over
local stdio and authenticated stateless Streamable HTTP.

Before pinning packages, inspect the current stable official MCP C# SDK and protocol documentation.
Record the reviewed version and transport behavior.

## MCP registration

Register exactly:

- `sap_plan_supplier_invoice`
- `sap_post_supplier_invoice`
- `sap_list_recently_paid_invoices`
- `sap_get_ontology_schema`
- resource `sap-invoice://ontology-schema`

Handlers map directly to the application operations from Stages 5–7. They must not construct SAP
URLs, accept raw OData, add generic tools, weaken limits, or bypass shared controls.

Mark annotations accurately:

- plan: non-read-only but no SAP mutation;
- post: mutating, non-idempotent, financially material;
- recently paid: read-only;
- schema tool and resource: read-only and idempotent.

Annotations are client hints, not authorization.

## Identity and authorization

For remote HTTP:

- use stateless Streamable HTTP at `/mcp`;
- require TLS at the deployment boundary;
- reject unauthenticated requests;
- validate bearer-token signature, issuer, audience, lifetime, and required claims locally;
- derive caller and tenant only from validated claims;
- configure exact allowed hosts and CORS origins; and
- reject spoofable forwarded identity headers from direct clients.

Use one deny-by-default authorization service with permissions:

- `sap.invoice.plan`
- `sap.invoice.post`
- `sap.invoice.payment.read`
- `sap.invoice.schema.read`

Invoice plan/post authorization also checks configured company code and amount boundary. Recent-paid
authorization filters company codes and supplier visibility. Denial must occur before any SAP token
or request.

For stdio, use an explicit local-development identity and fake SAP only. Refuse non-fake SAP
configuration in unauthenticated stdio mode.

## Bounded operation and telemetry

Enforce:

- MCP argument and message limits;
- request deadlines;
- per-caller concurrency and rate limits;
- bounded result sizes;
- graceful shutdown;
- read retries only when safe;
- zero invoice-post retries; and
- stable redacted errors.

Structured logs, metrics, traces, and audit events may contain correlation ID, pseudonymous actor,
tool name, company code when policy permits, outcome, duration, counts, and schema/config
fingerprints. They must not contain tokens, cookies, CSRF values, invoice bodies, supplier data,
purchase-order IDs, payment rows, SAP errors, or internal hosts.

## Tests

Add in-memory MCP tests proving:

- exactly four tools and one resource are advertised;
- stdio protocol output contains no logs;
- HTTP is authenticated stateless Streamable HTTP;
- invalid issuer, audience, signature, lifetime, or claims fail;
- permissions and company-code/amount boundaries are enforced;
- cross-caller plan execution fails;
- denied calls make no SAP or token-provider request;
- schema access makes no SAP request;
- rate, concurrency, cancellation, and shutdown are bounded; and
- no generic tool or raw SAP input appears in JSON Schemas.

## Acceptance criteria

- The MCP surface matches the four-tool contract exactly.
- Every tool uses validated identity and shared authorization.
- HTTP has no anonymous production path.
- Sensitive business data is redacted before telemetry sinks.
- Formatting, build, schema check, and tests succeed.

Commit locally. Use `narrative-required` and record the identity, authorization, and transport
decision. Do not push unless requested.
