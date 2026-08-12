# BrightFlagCFS and SAP Proxy MCP Prompt Assessment

> Reference snapshot converted from the supplied Word document dated 2026-07-29. The prompt sequence
> and implementation repositories may have evolved since this assessment; current prompts and
> repository instructions remain authoritative.

I have now reviewed the full staged prompt sequence in SAPProxyMCPServerBuilder and the current
state of SAPProxyMCPServer.

## What the Builder repository is designed to create

This is not intended to create a general-purpose SAP proxy. It is a deliberately constrained MCP
server with only three business capabilities:

1. Plan and post a purchase-order-referenced supplier invoice into SAP S/4HANA.

2. identify supplier invoices that have been fully cleared recently;

3. publish a deterministic ontology description of its tools and SAP concepts.

The completed server will expose exactly four MCP tools:

sap_plan_supplier_invoice sap_post_supplier_invoice sap_list_recently_paid_invoices
sap_get_ontology_schema

and one resource:

sap-invoice://ontology-schema

It explicitly prohibits arbitrary OData, BAPI, RFC, SOAP, ABAP, SQL and dynamic SAP service
discovery.

# Detailed assessment of the prompts

## Prompt 00 — Foundational contract

Prompt 00 establishes a very strong capability ceiling.

The server may interact with only two fixed SAP services:

API_SUPPLIERINVOICE_PROCESS_SRV API_OPLACCTGDOCITEMCUBE_SRV

The first is used for supplier-invoice creation. The second is used to obtain operational accounting
and clearing evidence.

Important safeguards include:

1. no caller-supplied SAP URL, entity, query or method;

2. no automatic retry of financial posting;

3. reviewed metadata rather than guessed SAP fields;

4. synthetic test data;

5. no credentials or financial payloads in Git or telemetry;

6. no functionality beyond the current prompt stage.

This is a sound foundation. It prevents the MCP server from becoming an uncontrolled route through
which an AI agent could invoke arbitrary SAP operations.

## Prompt 01 — Solution scaffold

Prompt 01 creates the .NET 10 solution and the stable business contracts, but deliberately does not
connect to SAP or MCP.

It defines:

1. the supplier-invoice draft;

2. posting plan and confirmation;

3. posted-invoice result;

4. recently paid query and result;

5. clearing evidence;

6. normalised SAP errors;

7. caller and tenant identity;

8. authorisation and audit results;

9. ontology schema.

It also correctly uses compound SAP identifiers:

Supplier invoice identity: (SupplierInvoice, FiscalYear) Accounting item identity: (CompanyCode,
FiscalYear, AccountingDocument, AccountingDocumentItem)

That is important because SAP document numbers are not necessarily globally unique without their
organisational and fiscal context.

## Prompt 02 — Project Narrative

Prompt 02 installs governance before allowing further functional decisions.

This is already reflected in the implementation repository. Narrative.md has been installed and
carries the correct title, although it currently contains no decision entries.

The repository correctly stops after this stage and requires the Narrative scaffold to be merged
before Prompt 03 begins. The README confirms this pause.

## Prompt 03 — SAP service configuration

Prompt 03 introduces the two fixed SAP services and the reviewed metadata snapshots.

Its strongest design elements are:

1. only one configured SAP origin per deployment;

2. optional SAP client;

3. approved company-code restrictions;

4. exact service roots;

5. externally supplied authentication references;

6. no secrets in configuration;

7. checked-in $metadata snapshots;

8. runtime never retrieves metadata dynamically;

9. failure if the target SAP release does not expose the required fields.

This is especially important because SAP OData services can differ by S/4HANA release, deployment
type and activated scope. The prompt correctly refuses to substitute a similarly named field when
the required one is absent.

## Prompt 04 — Invoice payload

Prompt 04 narrows version 1 to purchase-order-referenced supplier invoices only.

It includes:

1. company code;

2. document and posting dates;

3. invoicing party;

4. supplier invoice reference;

5. currency and gross amount;

6. automatic tax calculation;

7. optional text;

8. PO and PO item references;

9. quantity or amount;

10. tax code where required.

It explicitly excludes:

1. non-PO invoices;

2. G/L-only invoices;

3. assets;

4. material-account invoices;

5. one-time suppliers;

6. parking, holding and releasing;

7. arbitrary custom fields;

8. mixed currencies;

9. invalid posting dates;

10. unapproved tax codes.

The prompt also correctly states that the application must not attempt to reproduce SAP’s tax
engine. SAP remains responsible for actual tax calculation under the configured SAP tax rules.

## Prompt 05 — Plan and post

This is the most important security and financial-control prompt.

It separates invoice posting into:

Plan ↓ Trusted confirmation ↓ Post

The plan is:

1. random;

2. expiring;

3. single-use;

4. tied to the caller;

5. tied to the tenant;

6. tied to the company code;

7. tied to the normalised invoice;

8. tied to the active configuration and policy.

Posting then:

1. revalidates identity and authority;

2. consumes the plan atomically;

3. obtains SAP authentication;

4. acquires a CSRF token;

5. sends exactly one POST;

6. returns the SAP supplier invoice and fiscal year.

The prohibition on automatic retry is correct. When a connection is lost after SAP may have received
the invoice, retrying could create a duplicate financial document. The correct outcome is
indeterminate, followed by reconciliation.

## Prompt 06 — Recently paid invoices

This is also well considered.

The server does not define an invoice as paid merely because it is:

1. posted;

2. approved;

3. released;

4. due;

5. or not blocked.

Instead, it requires accounting clearing evidence:

1. supplier-payable item;

2. IsCleared;

3. clearing date;

4. clearing document;

5. reliable invoice or accounting-document linkage;

6. no remaining open payable item.

Partial clearing, residual open items, ambiguous linkage and reversed clearing are excluded.

This aligns much more closely with SAP FI-AP than a simple invoice-status query.

## Prompt 07 — Ontology publication

Prompt 07 creates a deterministic semantic catalogue of:

1. tools;

2. input and output schemas;

3. entities;

4. relationships;

5. authorisation requirements;

6. mutating and destructive characteristics;

7. SAP service provenance.

The ontology includes:

SupplierInvoice PurchaseOrderSupplierInvoiceDraft InvoicePostingPlan PostedSupplierInvoice
PaidSupplierInvoice Supplier PurchaseOrder ClearingDocument

This is highly relevant to the BrightFlagCFS repository because it provides a potential
machine-readable SAP boundary that the BrightFlag mapping or ontology capability could consume.

## Prompt 08 — Identity and authorisation

Prompt 08 applies deny-by-default permissions:

sap.invoice.plan sap.invoice.post sap.invoice.payment.read sap.invoice.schema.read

It also restricts plan and posting authority by:

1. tenant;

2. company code;

3. amount limit;

4. authenticated caller.

Remote MCP requires validated bearer tokens and stateless Streamable HTTP. Local stdio is restricted
to development identity and fake SAP only.

This is a strong design, although the actual identity provider and claims model will still need
alignment with the enterprise environment.

## Prompt 09 — Packaging and operations

Prompt 09 adds:

1. secure containerisation;

2. non-root operation;

3. CI;

4. fake SAP testing;

5. schema checks;

6. security scanning;

7. operations guidance;

8. threat modelling;

9. reconciliation documentation.

The threat model explicitly covers:

1. prompt injection;

2. confused deputy;

3. SSRF;

4. metadata poisoning;

5. credential leakage;

6. over-broad SAP authority;

7. invoice replay;

8. ambiguous posting;

9. false payment classification;

10. cross-tenant access;

11. ontology poisoning.

## Prompt 10 — Independent audit

Prompt 10 is a comprehensive reconstruction and adversarial audit containing 35 verification points.

It tests the completed server against:

1. capability boundaries;

2. posting controls;

3. payment-clearing evidence;

4. ontology accuracy;

5. identity and authorisation;

6. telemetry protection;

7. test isolation;

8. container and CI integrity.

It specifically tests attacks such as:

1. arbitrary URLs and OData;

2. plan replay;

3. cross-caller plan use;

4. CSRF theft;

5. non-PO payload injection;

6. false partial-payment classification;

7. clearing-date manipulation;

8. addition of a fifth tool or third SAP service.

# Current state of SAPProxyMCPServer

The implementation repository is accessible and currently contains:

1. the Stage 1 .NET 10 solution and contracts;

2. the Stage 2 Project Narrative installation;

3. no SAP connectivity;

4. no MCP endpoints;

5. no invoice posting;

6. no payment query.

The repository README explicitly says it is paused after Project Narrative and must not start Prompt
03 until that scaffold is published and merged.

The empty Narrative index confirms that no later decision-bearing implementation stage has yet been
recorded.

I found no current pull requests through the connected GitHub repository interface.

# Relevance to BrightFlagCFS

This server could provide a valuable prototype or reference implementation for BrightFlagCFS, but it
does not yet match the BrightFlag use case automatically.

The critical issue is:

SAPProxyMCPServer version 1 is exclusively designed for purchase-order-referenced invoices.

The BrightFlagCFS repository has not yet established whether legal invoices are:

1. PO-backed;

2. service-entry-sheet-backed;

3. non-PO G/L invoices;

4. parked invoices;

5. direct FI supplier invoices;

6. or another CFS-specific payment-authorisation construct.

Therefore, the Builder’s posting capability must not simply be inserted into BrightFlagCFS until
that question is answered.

## The parts that can be reused confidently

The following patterns are highly reusable:

1. narrow MCP tool surface;

2. fixed SAP services;

3. metadata snapshot and validation;

4. adapter isolation;

5. plan-before-post;

6. trusted confirmation;

7. no automatic mutation retry;

8. compound SAP identities;

9. clearing-based payment evidence;

10. bounded reconciliation query;

11. deterministic ontology schema;

12. deny-by-default company-code authority;

13. fake SAP test server;

14. adversarial audit.

## The parts that remain conditional

These require CFS confirmation:

1. API_SUPPLIERINVOICE_PROCESS_SRV availability;

2. API_OPLACCTGDOCITEMCUBE_SRV availability;

3. OData V2 activation;

4. purchase-order invoice process;

5. supported deep-create fields;

6. SAP client and company codes;

7. communication arrangement;

8. authentication mechanism;

9. tax policy;

10. full-clearing fields;

11. ability to link cleared operational items back to the originating supplier invoice.

# Important architectural observation

Despite the repository name, this is not genuinely a “proxy” in the general architectural sense.

It is better described as:

# A constrained SAP Supplier Invoice MCP Adapter

or:

# SAP Supplier Invoice and Clearing MCP Service

Calling it a proxy may imply arbitrary forwarding or generic access, which the design explicitly and
correctly prohibits.

# GitHub notifications

The connected GitHub capability available to me does not expose the personal GitHub notifications
inbox directly. I therefore could not inspect the bell-notification feed itself.

I did verify:

1. both repositories are accessible;

2. SAPProxyMCPServerBuilder is public;

3. SAPProxyMCPServer is private but readable;

4. there are no visible pull requests returned for the implementation repository;

5. the implementation repository currently remains at the Prompt 00–02 stage.

The most important next step is to validate whether the Narrative bootstrap actions have been
completed in the repository settings and then begin Prompt 03 as a separate decision-bearing change.
