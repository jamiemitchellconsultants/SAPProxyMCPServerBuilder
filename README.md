# SAPProxyMCPServerBuilder

A staged prompt sequence for building a narrowly scoped supplier-invoice MCP server with a coding
agent.

The completed server has three product capabilities:

1. Post a purchase-order-referenced supplier invoice through SAP S/4HANA OData V2 service
   `API_SUPPLIERINVOICE_PROCESS_SRV`.
2. Identify supplier invoices cleared recently using read-only operational accounting evidence from
   `API_OPLACCTGDOCITEMCUBE_SRV`.
3. Report a deterministic machine-readable schema describing its MCP tools, data types, provenance,
   and relationships so the server can be registered with an ontology server.

It is not a general SAP proxy. It cannot execute arbitrary OData, RFC, BAPI, SOAP, SQL, ABAP,
service-discovery, or metadata-browsing requests.

## New to GitHub or coding agents?

Start with [Start here: set up your repository and coding agent](START-HERE.md). It walks through
creating `MySAPProxyMCPServer`, cloning it locally, connecting one coding agent, installing the
development prerequisites, and proving the setup with a read-only task.

No production SAP access is needed to begin. Automated tests use a local fake SAP server and
synthetic invoice data.

## How to use the prompts

Submit the [reusable contract](prompts/00-reusable-contract.md) first. Then submit one implementation
prompt at a time only after the previous prompt passes its acceptance checks.

1. [Reusable contract](prompts/00-reusable-contract.md)
2. [Solution scaffold and invoice contracts](prompts/01-solution-scaffold-and-invoice-contracts.md)
3. [Adopt Project Narrative](prompts/02-adopt-project-narrative.md)
4. [Configure the two fixed SAP services](prompts/03-fixed-sap-service-configuration.md)
5. [Define and validate the supplier-invoice payload](prompts/04-supplier-invoice-payload-and-validation.md)
6. [Plan and post a supplier invoice](prompts/05-supplier-invoice-posting.md)
7. [Identify recently paid invoices](prompts/06-recently-paid-invoices.md)
8. [Report the ontology schema](prompts/07-ontology-schema-reporting.md)
9. [Expose the narrow MCP surface securely](prompts/08-mcp-identity-and-authorization.md)
10. [Package, document, and govern the server](prompts/09-delivery-documentation-and-audit.md)
11. [Run an independent reconstruction audit](prompts/10-independent-reconstruction-audit.md)
12. [Add home-lab fixed-token authentication](prompts/11-home-lab-fixed-token-authentication.md)
13. [Use the shared home-lab Keycloak OAuth flow](prompts/12-shared-home-lab-keycloak-oauth.md)
14. [Package the fake SAP upstream](prompts/12F-deployable-fake-sap-upstream.md)
15. [Prepare the `ai-mcp-server` deployment](prompts/13-ai-mcp-server-development-deployment.md)
16. [Add the SAP MCP deployment to LocalAI][p13l] — played in the LocalAI repository, not this one
17. [Audit the `ai-mcp-server` deployment contract](prompts/14-ai-mcp-server-deployment-audit.md)

Do not paste every prompt into one message. Each stage introduces one bounded capability and asks
for executable evidence before the next begins.

Prompts 11 to 14 deliberately override earlier rules: Prompt 11 adds the `File` secret source Prompt
3 did not establish, Prompt 12F allows the containerised fake to bind beyond loopback, and Prompt 13
replaces Prompt 3's plain-HTTP rule, enumerates the deployment profile Prompt 3 only alluded to, and
opens one unauthenticated readiness path against Prompt 8's blanket rejection. Each supersession is
stated in the prompt that makes it and audited by Prompt 14. Do not resolve a conflict between an
earlier and a later prompt by editing the earlier one.

Prompt 13L is the one exception to the repository location: play it in LocalAI after Prompt 13 has
merged into the server repository. It implements the host-owned deployment contract; do not copy
the LocalAI deployment script into the server repository.

[p13l]: https://github.com/jamiemitchellconsultants/LocalAI/blob/main/prompts/13L-add-sap-mcp-deployment-to-localai.md

Prompt 2 installs Project Narrative. Publish and merge that mechanical installation before opening
decision-bearing pull requests for later prompts.

> **Remember the second pull request.** A decision-bearing implementation pull request carries
> `narrative-required` and explicit Context, Decision, and Consequences. After it merges, Project
> Narrative creates a separate proposal containing the decision-history fragment. Review and merge
> that proposal without `narrative-required`.

## Exact MCP surface

The finished server exposes only:

- `sap_plan_supplier_invoice`
- `sap_post_supplier_invoice`
- `sap_list_recently_paid_invoices`
- `sap_get_ontology_schema`

It also exposes the read-only resource:

- `sap-invoice://ontology-schema`

The plan/post split is part of the posting capability, not a generic workflow engine. The posting
tool accepts only a server-issued, caller-bound, expiring plan for the fixed supplier-invoice deep
create.

## Capability boundaries

### Post a supplier invoice

- Uses OData V2 service `API_SUPPLIERINVOICE_PROCESS_SRV`.
- Posts only to the fixed `A_SupplierInvoice` entity set.
- Version 1 supports a reviewed purchase-order-referenced deep-create shape.
- Requires typed validation, balanced amounts, an explicit plan and confirmation, caller
  authorization, and SAP CSRF handling.
- Returns the created supplier invoice number and fiscal year.
- Does not release, reverse, delete, park, batch, or call arbitrary actions.

### Identify recently paid invoices

- Uses read-only service `API_OPLACCTGDOCITEMCUBE_SRV`.
- Treats an invoice as paid only when the supplier payable item has clearing evidence.
- Requires `IsCleared`, `ClearingDate`, and clearing-document evidence from reviewed metadata.
- Accepts a bounded lookback window: default 7 days, maximum 31 days.
- Returns a bounded projection suitable for reconciliation, not a bulk accounting export.
- Does not infer payment from supplier-invoice status alone.

### Report ontology schema

- Returns checked-in tool input/output JSON Schemas and semantic entity/relationship definitions.
- Describes `SupplierInvoice`, `InvoicePostingPlan`, `PaidSupplierInvoice`, `Supplier`,
  `PurchaseOrder`, and `ClearingDocument`.
- Includes source-service provenance and a deterministic fingerprint.
- Contains no live SAP data, credentials, hostnames, tokens, or caller-specific authorization.
- Does not let callers alter, upload, or execute ontology mappings.

## Deliberate exclusions

The server must not:

- accept an arbitrary base URL, OData path, entity, operation, query string, HTTP method, or header;
- publish tools dynamically from live `$metadata`;
- expose generic read, write, release, reverse, delete, park, batch, or action-import tools;
- support non-PO invoice shapes until a separately reviewed schema extension is added;
- treat “posted,” “released,” or “not blocked” as proof that an invoice was paid;
- scan unbounded accounting periods;
- retry invoice posting automatically;
- store SAP credentials, access tokens, CSRF tokens, cookies, or business payloads in Git or
  telemetry; or
- treat MCP descriptions, tool annotations, prompt text, or a model's assertion as authorization.

## Authoritative SAP references

- [Supplier Invoice — OData V2](https://help.sap.com/docs/SAP_S4HANA_CLOUD/bb9f1469daf04bd894ab2167f8132a1a/7bc52558ef790a02e10000000a44147b.html)
- [Supplier Invoice operations](https://help.sap.com/docs/SAP_S4HANA_CLOUD/bb9f1469daf04bd894ab2167f8132a1a/d8b16ace9227447c8c66086bc045a937.html)
- [Operational Journal Entry Item — Read](https://help.sap.com/docs/SAP_S4HANA_CLOUD/b978f98fc5884ff2aeb10c8fdeb8a43b/c024170fa7af40878975e218f3426387.html)
- [Operational accounting clearing fields](https://help.sap.com/docs/SAP_S4HANA_CLOUD/b978f98fc5884ff2aeb10c8fdeb8a43b/efabeff3bf8b4066ac03f0965d0a9327.html)
- [SAP CSRF token guidance](https://help.sap.com/docs/SAP_S4HANA_CLOUD/0f69f8fb28ac4bf48d2b57b9637e81fa/b886f94e821147eab21a90f5288a20e3.html)

The implementation prompts require checked-in metadata snapshots because exact fields and
communication arrangements vary by supported SAP release. Official metadata and reviewed
configuration—not examples—are authoritative.
