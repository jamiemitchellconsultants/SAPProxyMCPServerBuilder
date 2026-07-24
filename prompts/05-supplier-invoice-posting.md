# Prompt 5 — Plan and post a supplier invoice

Using the reusable contract, implement Stage 5: the plan-before-execute workflow and the single
allowed SAP mutation.

## Plan

Implement the application operation behind `sap_plan_supplier_invoice`.

It must:

- authorize the caller for the configured company code before acquiring SAP credentials;
- validate the closed version-1 PO invoice schema;
- normalize the invoice deterministically;
- produce a redacted human-readable summary;
- state that posting creates an SAP financial document and may be irreversible through this server;
- include gross amount, currency, supplier reference, PO references, and posting date;
- create a cryptographically random, expiring, single-use plan;
- bind the plan to caller, tenant, company code, normalized invoice hash, configuration fingerprint,
  and expiry; and
- return a server-authenticated confirmation challenge containing no credentials or sensitive
  payload.

Store only bounded expiring plan state behind an interface. Do not persist invoice plans to Git,
logs, telemetry, snapshots, or a general workflow database.

## Post

Implement the application operation behind `sap_post_supplier_invoice`.

It accepts only the plan ID and confirmation response. It must:

1. authenticate and authorize the same caller and tenant;
2. atomically consume the plan at most once;
3. reject expiry, replay, changed policy, changed identity, or changed arguments;
4. acquire SAP authentication outside tool arguments;
5. fetch an `X-CSRF-Token` from the fixed supplier-invoice service;
6. keep the CSRF token and session cookies scoped to that destination and execution;
7. send exactly one deep-create `POST` to
   `/sap/opu/odata/sap/API_SUPPLIERINVOICE_PROCESS_SRV/A_SupplierInvoice`;
8. parse and validate the response; and
9. return only the created supplier invoice number, fiscal year, approved status evidence,
   correlation ID, and audit ID.

Do not expose an arbitrary confirmation Boolean. A model's prose is not proof of human approval;
document which trusted client or gateway interaction supplies the confirmation response.

Never retry the POST. If the connection fails after the fake SAP server may have received the body,
return an indeterminate outcome with reconciliation instructions. Do not claim failure, rollback,
or safely retry.

## Explicit exclusions

The posting path must not call:

- `Post`, `Release`, `Cancel`, `$batch`, or another action import;
- another entity set;
- another service;
- a caller-supplied URL or header; or
- a second POST after an ambiguous outcome.

## Tests

Use fake clocks, identities, and the fake SAP server to prove:

- plan validation and authorization occur before credential or SAP access;
- plans are caller-, tenant-, payload-, company-code-, policy-, and expiry-bound;
- plan consumption is race-safe and single-use;
- CSRF acquisition and deep create share the destination and cookie session;
- the outbound body matches the validated mapping exactly;
- the success result contains the returned `SupplierInvoice` and `FiscalYear`;
- SAP validation errors are normalized and redacted;
- timeout after request receipt produces an indeterminate result and one POST total; and
- release, reverse, delete, park, batch, arbitrary action, and arbitrary entity attempts are
  impossible through public contracts.

## Acceptance criteria

- Only the fixed supplier-invoice deep-create mutation exists.
- Every post passes through authorization, validation, plan, confirmation, CSRF, single-use, and
  audit controls.
- No code path retries an invoice POST.
- Fake-server assertions prove exact method, path, headers, body, and call count.
- Formatting, build, and tests succeed.

Commit locally. Use `narrative-required` and record the plan/confirmation and no-retry decisions.
Do not push unless requested.
