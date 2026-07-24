# Prompt 7 — Report the ontology schema

Using the reusable contract, implement Stage 7: a deterministic machine-readable description of the
server for ingestion by an ontology server.

Generate and check in:

- `schema/tool-catalog.json`
- `schema/ontology-registration.json`
- `schema/schema.sha256`

Add:

```text
sap-invoice-mcp schema compile
sap-invoice-mcp schema check
```

`check` must fail without writing when checked-in artifacts are stale.

## Tool catalog

Describe exactly these tool contracts:

- `sap_plan_supplier_invoice`
- `sap_post_supplier_invoice`
- `sap_list_recently_paid_invoices`
- `sap_get_ontology_schema`

For every tool include:

- stable name and description;
- input and output JSON Schema;
- read-only, mutating, destructive, idempotent, and confirmation characteristics;
- required authorization permissions;
- possible stable error categories; and
- SAP source-service provenance.

Do not publish a generic SAP operation, metadata-derived tool, disabled operation, internal URL,
credential reference, or environment-specific value.

## Ontology registration

Create a versioned normalized document containing:

- server identity and schema version;
- deterministic fingerprint;
- entities:
  - `SupplierInvoice`
  - `PurchaseOrderSupplierInvoiceDraft`
  - `InvoicePostingPlan`
  - `PostedSupplierInvoice`
  - `PaidSupplierInvoice`
  - `Supplier`
  - `PurchaseOrder`
  - `ClearingDocument`
- stable properties, identifiers, types, nullability, sensitivity, and provenance;
- relationships:
  - a draft references one or more purchase orders;
  - a supplier issues a supplier invoice;
  - a posting plan proposes creation of a supplier invoice;
  - a posted result identifies a supplier invoice;
  - a paid supplier invoice is evidenced by a clearing document;
- the meaning of `fullyCleared`;
- source mappings to the two fixed SAP services; and
- explicit exclusions and confidence/evidence notes.

The schema reports semantics; it must not contain invoice instances, payment instances, suppliers,
purchase orders, credentials, hostnames, caller permissions, or live SAP metadata.

Align the exported catalog with the checked-in exported-MCP-catalog or normalized-entity input
contract used by the target ontology server. If the target contract is unavailable or incompatible,
retain the versioned neutral JSON documents and report the required adapter rather than inventing
compatibility.

## MCP access

Implement the application operation for `sap_get_ontology_schema` and the read-only resource
`sap-invoice://ontology-schema`. Both must return the same checked-in semantic content and
fingerprint without contacting SAP.

## Tests

Prove:

- generated schemas match runtime domain contracts;
- tool list is exactly four;
- entity and relationship IDs are stable and unique;
- every source mapping names one of the two fixed SAP services;
- repeated compilation is byte-identical;
- stale check never writes;
- tool and resource content match;
- schema access produces zero SAP and credential-provider calls; and
- seeded credentials, hosts, invoice data, and payment data cannot enter artifacts.

## Acceptance criteria

- The ontology server receives a deterministic, self-describing catalog.
- The schema describes contracts and provenance, never live business data.
- No fifth tool, arbitrary operation, or dynamic metadata capability appears.
- Formatting, build, schema check, and tests succeed.

Commit locally. Use `narrative-required` and record the chosen ontology registration contract. Do
not push unless requested.
