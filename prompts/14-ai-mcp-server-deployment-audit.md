# Prompt 14 — Audit the `ai-mcp-server` deployment contract

Using the reusable contract and the artifacts produced by Prompts 1–13, independently audit the
completed server and the LocalAI implementation produced by Prompt 13L.

This is an evidence and correction stage. It does not authorise deployment, live Keycloak mutation,
SAP metadata refresh, SAP credential use or a supplier-invoice POST. Inspect LocalAI read-only from
the sibling repository or from a supplied clean checkout. If it is unavailable, complete the server
checks and report the cross-repository checks as blocked rather than guessing.

## Reconstruct the invariant surface

Prove from registration, schemas and executable tests that the server still exposes exactly:

- `sap_plan_supplier_invoice`;
- `sap_post_supplier_invoice`;
- `sap_list_recently_paid_invoices`;
- `sap_get_ontology_schema`; and
- resource `sap-invoice://ontology-schema`.

No deployment setting may introduce a third SAP service, arbitrary origin, path, entity, action,
method, header, query string, raw OData input, metadata-driven tool or fifth tool.

Re-run every Prompt 10 invariant, including plan binding and consumption, trusted confirmation,
zero automatic POST retries, clearing evidence, bounded reads, deterministic ontology output,
denial-before-SAP and telemetry redaction.

## Audit authentication and OAuth discovery

Prove:

- exactly one of fixed token or Keycloak is constructed and there is no fallback;
- fixed token is file-backed, constant-time compared, fully redacted and establishes one configured
  caller;
- fixed-token 401 responses are bare Bearer challenges and serve no metadata;
- Keycloak validates signature, algorithm, issuer, `homelab-mcp`, lifetime, `tid` and flat roles;
- both OAuth metadata paths return one document and optional scope behavior is exact;
- a dynamically registered client identifier grants no authority;
- all four SAP permissions remain enforced through the deny-by-default authorization service; and
- company-code, amount, supplier-visibility, tenant and caller-bound plan controls survive in both
  authentication modes.

## Audit the cross-repository configuration contract

Build the exact candidate image and read its configuration manifest without starting HTTP. Generate
the LocalAI Compose model without exposing secret values, then compare it mechanically with that
manifest.

Fail for:

- an unknown generated setting;
- a missing required or conditionally required setting;
- a secret supplied through an environment literal, command argument, Docker label or state file;
- a mode-incompatible setting;
- an unsupported upstream class; or
- a source-revision mismatch between resolved commit, image label, manifest and deployment state.

Repeat the comparison against the retained rollback image. A rollback that cannot describe its own
contract is refused rather than attempted with current configuration.

## Audit the generated topology

Without changing the live host, prove from generated artifacts that:

- one SAP server container joins external `mcp-public` and publishes no host port;
- its class is `Fake` or `Sandbox`, never `Production`;
- its health check sends one of the three allowed Host values;
- one SAP-owned Caddy fragment serves `sap-invoice-mcp.tqaentry.com`, disables response buffering
  with `flush_interval -1`, and changes no other fragment or root Caddyfile;
- shared Caddy is a hard precondition and full-config validation precedes reload;
- failed validation or reload restores the previous fragment and fails the operation;
- Keycloak discovery is read-only in the SAP deployment script;
- LocalStack values are materialised atomically into ACL-protected files and are absent from
  Compose, logs, arguments, Docker metadata and state;
- successful state is recorded only after health passes;
- rollback selects the recorded image identity without resolving or rebuilding a Git ref; and
- diagnostics, stop and removal check command results and affect only SAP-owned resources.

## Audit shared Keycloak reconciliation

Inspect LocalAI's desired-state logic and prove that `mcp-roles` is read back and corrected when its
protocol mapper, claim name, JSON type, token flags or value differs. A matching name alone is not
evidence.

Verify the shared role value contains the four SAP permissions as well as every previously supported
ontology and BrightFlag role. Verify required capability client scopes are realm defaults and that
the documentation states what happens to clients registered before a new default scope existed.

Do not require a token to contain only the named claims; Keycloak access tokens legitimately contain
subject, lifetime and other protocol claims. Assert presence and values of the security-relevant
claims instead.

## Manual gates

List, but do not silently execute, the exact PowerShell 7.6 commands and expected results for:

- generated Compose and Caddy inspection;
- Caddy validation and reload;
- public DNS and certificate validation;
- Keycloak discovery from inside the container;
- dynamic registration and authorization-code with PKCE;
- fixed-token authentication;
- one read-only fake or sandbox SAP call;
- direct-backend refusal and Streamable HTTP non-buffering;
- exact-commit upgrade and image-identity rollback; and
- host and Docker Desktop restart recovery.

A live supplier-invoice POST is a separately authorised financial operation and is never an audit
prerequisite. Do not report any unexecuted host or SAP check as passing.

## Acceptance criteria

- Every Prompt 10 application invariant still passes after the deployment stages.
- Server and LocalAI configuration agree mechanically through the exact image manifest.
- The generated topology has one HTTPS entry point and no direct backend exposure.
- Both authentication modes work without fallback and without authorization weakening.
- The home-lab deployment cannot select production SAP.
- No secret or sensitive business value crosses a repository, generated-artifact or telemetry
  boundary.
- Every automated check passes, and every unavailable environmental check is an exact manual gate.

Correct defects only within the repository that owns them and preserve unrelated work. Commit any
correction locally with `narrative-required` and the required narrative sections. Do not push or
deploy unless requested.
