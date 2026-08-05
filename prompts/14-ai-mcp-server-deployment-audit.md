# Prompt 14 — Audit the `ai-mcp-server` deployment contract

Using the reusable contract and the artifacts produced by Prompts 1–13, including 12F, independently
audit the completed server and the LocalAI implementation produced by Prompt 13L.

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

## Audit the deliberately superseded rules

Prompts 11, 12F and 13 replace four earlier rules. Each replacement is legitimate only if it is
recorded, bounded, and not quietly wider than stated. Prove:

- the `File` secret source from Prompt 11 reaches every secret kind Prompt 3 defined, refuses a
  literal value in a path position, and leaves no environment-variable route to a secret-classified
  setting in any mode;
- the fake provider is refused under `Sandbox` and `Production`;
- plain HTTP is permitted for `Fake` only, and the Prompt 3 loopback-only wording is marked
  superseded rather than left standing beside its replacement;
- the containerised fake binds `0.0.0.0` while the in-process test double still binds loopback, and
  neither serves a route the other does not;
- exactly one unauthenticated path exists beyond Prompt 12's two metadata paths, it is `/health`,
  and reaching `/mcp`, any tool, any resource, any administrative verb or any error detail without a
  credential remains impossible in both modes; and
- every other Prompt 3 URL rejection — caller-controlled origins and paths, user information,
  fragments, wildcards, traversal, query strings, cross-origin redirects, third services — is
  intact and still fails closed.

Report any rule that changed without an explicit supersession record as a defect, even where the new
behaviour is the safer one.

## Audit the fixed image contract

Prove mechanically, against the built candidate image and against the LocalAI script's source text,
that both repositories use the same strings:

- build argument `SOURCE_REVISION`;
- label `org.opencontainers.image.revision`;
- assembly path `/app/SapInvoiceMcpServer.dll`; and
- the manifest command, entry-point override included.

A LocalAI script that hardcodes a different assembly name, omits the entry-point override, derives
environment-variable names by string transformation instead of reading the manifest's declared form,
or parses the manifest command's output with a tolerant fallback is a defect. So is a server image
whose entry point makes the administrative verb unreachable.

Confirm the fake-upstream image carries the same argument and label, a title that identifies it as a
fake, and no outbound HTTP client, upstream origin or credential anywhere in its source.

## Audit authentication and OAuth discovery

Prove:

- exactly one of fixed token or Keycloak is constructed and there is no fallback;
- fixed token is file-backed, constant-time compared, fully redacted and establishes one configured
  caller;
- fixed-token 401 responses are bare Bearer challenges and serve no metadata;
- Keycloak validates signature, algorithm, issuer, `homelab-mcp`, lifetime, `tid` and flat roles;
- both OAuth metadata paths return one document, the advertised scope is exactly
  `sap.invoice.schema.read`, and that name exists in the realm as a default client scope;
- a dynamically registered client identifier grants no authority;
- each role value grants exactly its identically named permission, with no alias table, prefix rule
  or hierarchy, and an unrecognised role value grants nothing;
- the accepted tenant is required configuration in both modes, a validated token bearing another
  `tid` is refused, and an absent value fails startup;
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
- its class is `Fake` or `Sandbox`, never `Production`, and its profile is `NonProduction`;
- its health check probes `/health`, sends one of the three allowed Host values, and matches on the
  readiness token rather than on an HTTP status alone;
- a selected fake upstream runs as its own container on a private SAP-owned network, is not joined
  to `mcp-public`, publishes no host port, and is absent entirely when the class is `Sandbox`;
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
ontology and BrightFlag role. Verify required capability client scopes are realm defaults, that
`sap.invoice.schema.read` is among them because the server advertises it, and that the documentation
states what happens to clients registered before a new default scope existed.

Verify the tenant the realm mints in `tid` is the same value the generated SAP configuration
declares as its accepted tenant. A deployment where those two disagree authenticates successfully
and then denies every call, which is the failure mode this check exists to catch.

Do not require a token to contain only the named claims; Keycloak access tokens legitimately contain
subject, lifetime and other protocol claims. Assert presence and values of the security-relevant
claims instead.

## Manual gates

List, but do not silently execute, the exact PowerShell 7.6 commands and expected results for:

- generated Compose and Caddy inspection;
- the manifest command against the built candidate image, and the label read-back that proves the
  commit;
- Caddy validation and reload;
- public DNS and certificate validation;
- unauthenticated `/health` through the public endpoint, and an unauthenticated `/mcp` refusal
  beside it;
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
- Every rule Prompts 11, 12F and 13 superseded is recorded as superseded, and no rule changed
  without one.
- Server and LocalAI configuration agree mechanically through the exact image manifest, and both
  repositories use the same build-argument, label, assembly-path and command-line strings.
- The generated topology has one HTTPS entry point, no direct backend exposure, and exactly one
  unauthenticated path.
- Both authentication modes work without fallback and without authorization weakening, and the
  realm's tenant and role values match what the server accepts.
- The home-lab deployment cannot select production SAP.
- No secret or sensitive business value crosses a repository, generated-artifact or telemetry
  boundary.
- Every automated check passes, and every unavailable environmental check is an exact manual gate.

Correct defects only within the repository that owns them and preserve unrelated work. Commit any
correction locally with `narrative-required` and the required narrative sections. Do not push or
deploy unless requested.
