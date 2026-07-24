# Prompt 4 — Define and validate the supplier-invoice payload

Using the reusable contract, implement Stage 4: the closed version-1 payload for a
purchase-order-referenced supplier-invoice deep create.

Map a domain `PurchaseOrderSupplierInvoiceDraft` to the reviewed
`API_SUPPLIERINVOICE_PROCESS_SRV` metadata. The supported request includes only the reviewed fields
needed for:

- company code;
- document date and posting date;
- invoicing party;
- supplier's invoice reference;
- document currency;
- gross amount;
- automatic tax calculation under an approved tax policy;
- optional permitted header text;
- one or more purchase-order reference lines;
- line number, purchase order and item;
- quantity/unit or amount fields required by the reviewed SAP shape;
- tax code when required; and
- the PO-reference navigation used by the SAP deep create.

Use exact SAP wire names from the checked-in metadata snapshot. Keep those names inside the SAP
adapter; the MCP/domain schema uses clear stable names.

Version 1 must not accept:

- caller-selected invoice status;
- held, parked, release, reverse, delete, or batch instructions;
- G/L-only, asset, material-account, one-time-supplier, or non-PO line shapes;
- arbitrary custom fields;
- raw SAP navigation properties;
- negative or zero invoice totals;
- mixed currencies;
- duplicate line numbers;
- missing PO references;
- dates outside a configured posting window;
- unapproved company code, currency, tax code, or amount boundary; or
- extra JSON properties.

Do not claim to reproduce SAP tax calculation. Validate structural and configured business
boundaries, then let SAP apply its reviewed tax rules. Return a field-level validation report before
any plan can be created.

Add deterministic JSON Schema for the draft and result types. Examples must be synthetic and must
not contain a real supplier, purchase order, invoice number, company code, hostname, or amount.

## Tests

Prove:

- a valid one-line and multi-line PO invoice maps to the exact expected SAP deep-create body;
- unknown, generic, non-PO, and custom fields are rejected;
- company code, dates, amounts, currency, tax, line uniqueness, and PO references are bounded;
- JSON decimal serialization is culture-independent and does not lose precision;
- no validation path sends an SAP request; and
- snapshots redact or replace business identifiers.

## Acceptance criteria

- The supported invoice schema is closed and versioned.
- Only the PO-reference deep-create navigation can be emitted.
- Validation is deterministic and network-free.
- The exact SAP body is proven against the fake server contract.
- Formatting, build, and tests succeed.

Commit locally. Use `narrative-required` and record the decision to start with one PO-backed invoice
shape rather than a generic supplier-invoice payload. Do not push unless requested.
