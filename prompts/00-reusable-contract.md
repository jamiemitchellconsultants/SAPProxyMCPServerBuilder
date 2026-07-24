# Reusable contract

Submit this contract before the implementation prompts. If stages run in separate coding-agent
tasks, prepend this contract to every stage.

You are creating a production-quality repository called `SAPProxyMCPServer` in an empty or
partially initialized workspace.

## Product objective

Build a narrowly scoped MCP server with exactly three business capabilities:

1. Plan and post a purchase-order-referenced supplier invoice through SAP S/4HANA OData V2 service
   `API_SUPPLIERINVOICE_PROCESS_SRV`.
2. List supplier invoices cleared within a recent bounded period using read-only clearing evidence
   from `API_OPLACCTGDOCITEMCUBE_SRV`.
3. Report a deterministic schema that an ontology server can ingest to understand the MCP tools,
   entities, relationships, field types, and SAP provenance.

The server is not a general SAP proxy or generic OData client.

## Exact MCP contract

Expose only:

- `sap_plan_supplier_invoice`
- `sap_post_supplier_invoice`
- `sap_list_recently_paid_invoices`
- `sap_get_ontology_schema`

Expose one read-only resource:

- `sap-invoice://ontology-schema`

Do not dynamically add tools or resources from SAP metadata.

## Non-negotiable architecture

- Use the latest supported patch of .NET 10 LTS, C#, ASP.NET Core, and the stable official MCP C#
  SDK.
- Support local stdio and remote authenticated stateless Streamable HTTP at `/mcp`.
- Pin centrally managed NuGet versions after checking current primary documentation.
- Configure exactly two SAP OData V2 service roots:
  - `/sap/opu/odata/sap/API_SUPPLIERINVOICE_PROCESS_SRV`
  - `/sap/opu/odata/sap/API_OPLACCTGDOCITEMCUBE_SRV`
- Never accept a base URL, path, entity, operation, query string, HTTP method, header, or credential
  from an MCP argument.
- Version 1 posts only purchase-order-referenced invoices using deep create on
  `A_SupplierInvoice`.
- Do not expose release, reverse, delete, park, hold, batch, G/L-only, asset, material, one-time
  supplier, or arbitrary action operations.
- Invoice posting requires typed validation, caller authorization, plan-before-execute
  confirmation, CSRF handling, and a single outbound POST with no automatic retry.
- “Recently paid” means a supplier payable item carries reviewed clearing evidence, including
  `IsCleared`, `ClearingDate`, and clearing-document data. Invoice status alone is not payment
  evidence.
- The lookback defaults to 7 days, has a maximum of 31 days, and always uses bounded filters,
  projections, paging, and response limits.
- The ontology schema is generated deterministically from checked-in contracts. It contains no live
  SAP data or environment-specific values.
- Credentials, access tokens, cookies, CSRF tokens, certificates, invoice payloads, supplier data,
  and payment records never enter Git or telemetry.
- Use synthetic fixtures and a local fake SAP server for automated tests.
- Live sandbox tests are opt-in and excluded from normal CI.
- Do not add RFC, BAPI, SOAP, SQL, ABAP, browser automation, generic OData, or other SAP
  capabilities.
- Do not implement functionality beyond the current stage.

## Working method

1. Inspect the workspace and repository instructions before editing.
2. Explain the current stage and state a short plan.
3. Verify unstable SDK and SAP field details against official documentation and reviewed metadata.
4. Implement the smallest coherent increment.
5. Add success, rejection, authorization, and boundary tests.
6. Assert the exact outbound method, path, query, headers, body, and call count observed by the fake
   SAP server.
7. Run formatting, build, tests, schema checks, and security checks; fix failures.
8. Report files changed, commands, results, assumptions, and remaining risks.
9. Commit locally when asked, but do not push, open, label, or merge a pull request unless explicitly
   requested.

## Evidence rules

- Acceptance criteria require executable evidence.
- Do not weaken validation, authorization, confirmation, or tests to make a stage pass.
- If reviewed SAP metadata does not contain a required posting or clearing field, stop and report
  the incompatibility instead of guessing a field name or redefining “paid.”
- Tool descriptions, annotations, prompt text, and a model's claim never grant authority.
