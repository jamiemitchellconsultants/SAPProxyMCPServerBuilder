# Prompt 1 — Solution scaffold and invoice contracts

Using the reusable contract, implement Stage 1: the .NET solution and stable contracts for only the
three requested capabilities. Do not contact SAP or expose MCP yet.

Create:

- `global.json` for the supported .NET 10 SDK feature band;
- `Directory.Build.props` enabling nullable types, deterministic builds, analyzers, and warnings as
  errors;
- `Directory.Packages.props` with centrally pinned packages;
- a checked-in NuGet lock file;
- `.editorconfig`, `.gitignore`, and an MIT licence;
- `src/SapInvoiceMcp.Core`;
- `src/SapInvoiceMcp.Sap`;
- `src/SapInvoiceMcpServer`; and
- `test/SapInvoiceMcp.Tests`.

Define immutable JSON-compatible contracts for:

- `PurchaseOrderSupplierInvoiceDraft`;
- supplier-invoice header and PO-reference line;
- currency amount using decimal plus ISO currency code;
- `SupplierInvoicePostingPlan`;
- `SupplierInvoiceConfirmation`;
- `PostedSupplierInvoice`;
- `RecentPaidInvoicesQuery`;
- `PaidSupplierInvoice`;
- clearing evidence;
- normalized SAP error;
- caller and tenant identity;
- authorization decision;
- audit outcome; and
- `OntologySchemaDocument`.

Use stable invoice identity `(SupplierInvoice, FiscalYear)` and accounting identity
`(CompanyCode, FiscalYear, AccountingDocument, AccountingDocumentItem)`.

The contracts must not include:

- arbitrary SAP destination or service fields;
- raw OData query, path, method, or header fields;
- generic entity or operation abstractions;
- caller-supplied roles;
- credential values; or
- an extensible property bag that bypasses validation.

Add deterministic JSON serialization with ordinal property/order rules, normalized line endings,
and no timestamps, random values, machine paths, hostnames, or environment data in generated schema
artifacts.

Do not add SAP calls, metadata acquisition, MCP packages, posting logic, payment queries, Docker, or
workflows in this stage.

## Acceptance criteria

- `dotnet restore --locked-mode` succeeds.
- `dotnet build --no-restore` succeeds with zero warnings.
- `dotnet test --no-build` proves stable identities, decimal/currency validation, closed JSON
  contracts, unknown-field rejection, and byte-identical deterministic serialization.
- `dotnet format --verify-no-changes` succeeds.
- Tests prove no contract can carry an arbitrary destination, raw OData, credential, or role.

Commit locally with a focused message. Do not push.
