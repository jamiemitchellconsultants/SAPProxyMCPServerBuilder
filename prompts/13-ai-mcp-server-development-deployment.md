# Prompt 13 — Prepare the `ai-mcp-server` development deployment

Using the reusable contract and the artifacts produced by Prompts 1–12, prepare
`SAPProxyMCPServer` for one development deployment on `ai-mcp-server` and hand executable deployment
ownership to the separately managed LocalAI repository.

This stage does not deploy anything. It changes the server's deployment contract, transport
configuration and documentation. Prompt 13L is played separately in LocalAI after this stage and
implements the host-side script. Prompt 14 audits both results.

The deployment is for a physically controlled home lab. It is not a production SAP deployment and
must never be presented as one.

## Declare the upstream class

Extend the single SAP-origin configuration from Prompt 3 with one required closed upstream class
and no default:

- `Fake` — a non-SAP test double on a loopback, private or container-network address;
- `Sandbox` — a reviewed non-production SAP system over HTTPS; and
- `Production` — a production SAP system over HTTPS.

Validate origin, profile and secret provider against the declared class:

| Class | Address and scheme | Credential posture | Profile |
|---|---|---|---|
| `Fake` | HTTP(S), private only | fake allowed | non-production only |
| `Sandbox` | HTTPS, reviewed SAP | external real secret | reviewed sandbox only |
| `Production` | HTTPS, reviewed SAP | production secret | production only |

All Prompt 3 URL and closed-service rejections remain. The server cannot prove what actually answers
at an origin, so document that the class is a reviewed declaration rather than remote attestation.

Under `Fake` and `Sandbox`, every audit event, log scope and tool response body carries an
unmistakable marker naming the class. Include a successful supplier-invoice posting response in the
proof. Under `Production`, do not add a warning marker.

The LocalAI deployment accepts only `Fake` or `Sandbox`. Its parameter type must be incapable of
expressing `Production`; a comment is not an enforcement boundary.

## Generate the cross-repository configuration contract

Add an administrative command to the shipped server image that emits a deterministic,
machine-readable manifest derived from the actual options tree. It contains no configuration or
secret values. For every accepted setting it reports:

- canonical name and environment-variable form;
- value type and collection shape;
- required, optional or conditionally required status;
- supported enum values and relevant mode conditions;
- whether it is secret material, a secret identifier, a file path or ordinary configuration; and
- whether unknown settings in that options subtree fail startup.

The manifest also reports the source revision embedded in the image and a manifest schema version.
It must be available without starting the HTTP listener or contacting SAP, Keycloak or a secret
provider. Unknown command arguments fail.

LocalAI will ask the exact candidate image for this manifest before deployment and rollback. Do not
maintain a second hand-written list of setting names and do not teach LocalAI to infer names from
source code.

## Serve the canonical endpoint through shared Caddy

The canonical development endpoint is:

`https://sap-invoice-mcp.tqaentry.com/mcp`

Support an explicit trusted-proxy transport posture selected from reviewed configuration. For this
deployment:

- shared Caddy terminates public TLS;
- the container listens on internal `http://0.0.0.0:<port>` only;
- the server's configured external resource URI is HTTPS;
- no host port is published for the server container;
- the server does not process forwarded caller identity or forwarded client-address headers; and
- the trusted-proxy name is a deployment declaration, not a value a caller can supply.

Host validation permits exactly:

- `sap-invoice-mcp.tqaentry.com`;
- `localhost`; and
- the container alias used by the Compose health check.

Anything else is refused. The health check sends an allowed Host value. CORS remains an explicit
configured allow-list and is not broadened to `*` for conversational agents.

## Make LocalAI the sole development deployment owner

Do not add a Compose project, Caddy template, Windows deployment script or LocalStack client to this
repository. If an earlier server-owned development deployment exists, retire it and mark every
retained historical reference superseded and inert.

Document that LocalAI's `prompts/13L-add-sap-mcp-deployment-to-localai.md` creates and governs
`docs/setup-sap-mcp-windows.ps1`. The server repository owns:

- its container image and source-revision label;
- its configuration manifest and startup validation;
- transport, host, authentication and authorization behavior; and
- documentation of what a conforming deployment must supply.

LocalAI owns Git-ref resolution, exact-commit building, Compose, Caddy, host-side secret
materialisation, deployment state, health gating, upgrade, rollback, stop, removal and diagnostics.

## Require explicit deployment inputs

The deployment contract must identify, without inventing values:

- SAP origin and class;
- optional SAP client number;
- SAP credential mechanism and every required file secret;
- allowed company codes and the posting amount/currency policy;
- fixed-token file, subject and tenant when fixed-token mode is selected;
- Keycloak issuer, audience, resource URI and optional advertised scope in Keycloak mode;
- every plan, cursor or signing secret introduced by earlier stages;
- exact allowed hosts, CORS origins, limits and timeouts; and
- the container alias and health path.

No SAP origin, company code, amount, currency, client number, credential identifier or advertised
scope receives a plausible default. File secrets stay behind application-level file-provider
contracts; no LocalStack or AWS SDK enters application code.

## Prove

- upstream class is required and every invalid class, scheme, address, profile and provider pairing
  fails startup;
- LocalAI's supported class set cannot express `Production`;
- fake and sandbox responses, including successful post results, carry their class marker;
- the administrative manifest is deterministic, value-free, derived from the bound options,
  available offline and labelled with the image source revision;
- the trusted-proxy posture requires and accepts the HTTPS external URI while Kestrel remains
  internal HTTP;
- host validation accepts exactly the three configured names, including the healthcheck alias;
- both authentication modes work under the topology and retain their distinct challenges;
- no server-owned deployment entry point or vendor secret provider is introduced; and
- the complete four-tool, one-resource surface and two fixed SAP services remain unchanged.

## Acceptance criteria

- The server image exposes a self-describing configuration contract that LocalAI can validate before
  replacing a running container.
- The documented deployment is public HTTPS through shared Caddy with no direct backend listener.
- The LocalAI development deployment is structurally unable to select production SAP.
- Every environment-specific and financially material value is explicit rather than guessed.
- Prompt 13L is identified as the separate deployment implementation stage.
- Formatting, build, schema checks, container build and tests succeed.

Commit locally. Use `narrative-required` and record the upstream-class declaration, non-production
marking, generated cross-repository contract, final HTTPS trusted-proxy topology and transfer of
deployment ownership to LocalAI. Do not push unless requested and do not deploy to the host.
