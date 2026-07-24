# Prompt 6 — Identify recently paid invoices

Using the reusable contract, implement Stage 6: bounded read-only identification of supplier
invoices that were fully cleared recently.

Use only:

```text
/sap/opu/odata/sap/API_OPLACCTGDOCITEMCUBE_SRV
```

and the reviewed operational accounting item entity from its checked-in metadata.

## Definition of recently paid

For this server, “paid” means all supplier-payable operational accounting items belonging to the
invoice are cleared and carry clearing evidence. At minimum:

- the item is a supplier/payables item according to reviewed metadata;
- `IsCleared` is true;
- `ClearingDate` falls inside the requested inclusive lookback window;
- the clearing accounting document is present; and
- the item links to a stable supplier-invoice or source accounting-document identity.

Do not infer payment from supplier-invoice posting, release, block, due, or document status.
Exclude partially cleared invoices, residual open items, ambiguous invoice linkage, reversals that
invalidate the clearing evidence, and records outside the window.

If the target SAP release cannot prove full clearing and invoice linkage from the reviewed API
fields, stop and report the incompatibility rather than weakening the definition.

## Tool query

Implement the operation behind `sap_list_recently_paid_invoices`.

Its closed input contains only:

- `lookbackDays`, optional, default `7`, minimum `1`, maximum `31`;
- `companyCode`, optional only when within the caller's authorized configured set; and
- `maxResults`, optional, default `50`, maximum `100`.

Do not accept raw filters, dates, OData, sort expressions, fields, entity names, URLs, or page
tokens from the caller.

Calculate the inclusive date boundary using a configured business timezone and an injectable clock.
Compile the exact OData V2 filters and projection server-side. Query recently cleared candidate
supplier items, then perform a bounded verification that no payable item for a candidate invoice
remains open. Bound candidate count, verification queries, pages, rows, bytes, duration, and
concurrency.

The result for each invoice contains only reviewed fields:

- supplier invoice and fiscal year, when authoritatively linked;
- company code;
- source accounting document identity;
- supplier identifier only when caller policy permits it;
- transaction amount and currency;
- clearing date;
- clearing accounting document and fiscal year;
- `fullyCleared: true`;
- evidence summary and source service; and
- correlation ID.

Sort deterministically by clearing date descending, then stable invoice/accounting identity.
Deduplicate multiple payable items into one invoice result only after full-clearing verification.

## Tests

The fake SAP server must cover:

- fully cleared invoice;
- partially cleared invoice;
- open invoice;
- cleared non-supplier item;
- clearing just inside and outside the window;
- missing clearing document;
- ambiguous invoice linkage;
- reversed or invalidated clearing;
- multiple payable items for one invoice;
- unauthorized company code;
- result/page/byte limits; and
- malicious SAP error or continuation payload.

Assert the exact fixed path, generated filters, selected fields, ordering, pages, and GET-only call
count. Prove that no payment record or supplier data enters logs or traces.

## Acceptance criteria

- “Paid” is backed by full-clearing evidence, not an invoice status guess.
- Lookback is 1–31 days and results are capped at 100.
- Every query is generated server-side for the one fixed read service.
- Partial, open, ambiguous, and invalidated records are excluded.
- Formatting, build, and tests succeed.

Commit locally. Use `narrative-required` and record the definition of recently paid and its evidence
limitations. Do not push unless requested.
