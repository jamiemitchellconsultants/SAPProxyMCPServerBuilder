# Prompt 13 — Prepare the `ai-mcp-server` development deployment

Using the reusable contract and the artifacts produced by Prompts 1–12 and 12F, prepare
`SAPProxyMCPServer` for one development deployment on `ai-mcp-server` and hand executable deployment
ownership to the separately managed LocalAI repository.

This stage does not deploy anything. It changes the server's deployment contract, transport
configuration and documentation. Prompt 13L is played separately in LocalAI after this stage and
implements the host-side script. Prompt 14 audits both results.

The deployment is for a physically controlled home lab. It is not a production SAP deployment and
must never be presented as one.

## What this stage supersedes

Three earlier rules are replaced, not reinterpreted. Record each replacement explicitly in the
delivered documentation and mark the superseded wording inert:

| Superseded | Replacement |
|---|---|
| Plain HTTP permitted only for the loopback fake server (Prompt 3) | Plain HTTP permitted only for the `Fake` class, on a loopback, private or container-network address |
| Deployment profile referred to only as "production" versus not (Prompt 3) | One closed two-value profile, defined below, that the upstream class is validated against |
| Every remote-HTTP request is rejected unauthenticated (Prompt 8) | One fixed unauthenticated readiness path, defined below, and nothing else |

Every other Prompt 3 URL rejection stands unchanged: caller-controlled origins or paths, URL user
information, fragments, wildcards, traversal, query strings, cross-origin redirects, third services,
credentials in JSON or URLs, and unknown configuration properties.

## Define the deployment profile

Prompt 3 mentioned production profiles without enumerating them. Add one required closed profile
with no default and exactly two values:

- `NonProduction`; and
- `Production`.

The profile is reviewed startup configuration, independent of the upstream class, and is what the
class table below validates against. An absent, unknown or ambiguous profile fails startup by name
before the transport accepts a request. Do not derive it from a hostname, an environment-name
convention, a build configuration or the upstream class itself.

## Declare the upstream class

Extend the single SAP-origin configuration from Prompt 3 with one required closed upstream class
and no default:

- `Fake` — a non-SAP test double, packaged by Prompt 12F;
- `Sandbox` — a reviewed non-production SAP system; and
- `Production` — a production SAP system.

Validate origin, profile and secret source against the declared class, in both directions:

| Class | Scheme | Address | Secret source | Profile |
|---|---|---|---|---|
| `Fake` | HTTP or HTTPS | loopback, private or container-network only; publicly routable refused | fake or `File`; no real SAP credential required | `NonProduction` only |
| `Sandbox` | HTTPS only | publicly routable or reviewed private SAP; loopback refused | `File` only; fake provider refused | `NonProduction` only |
| `Production` | HTTPS only | publicly routable only; loopback, private and container-network refused | `File` only; fake provider refused | `Production` only |

Every cell is a startup check with its own failure message naming the setting and the class that
made it invalid. A publicly routable `Fake`, a loopback `Sandbox`, an HTTP `Sandbox`, a private
`Production`, a fake-sourced `Sandbox` and a `Production` class under a `NonProduction` profile all
fail, and each fails distinctly.

The server cannot prove what actually answers at an origin, so document that the class is a reviewed
declaration rather than remote attestation.

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

## Fix the image contract by name

LocalAI has to write these strings into a script before it can run the image once. They are a
cross-repository contract, not an implementation choice, and changing one is a breaking change to
the deployment:

- the build argument carrying the commit is `SOURCE_REVISION`, defaulting to `unknown`;
- the image records it as OCI label `org.opencontainers.image.revision`, alongside a title,
  description, source and licence label;
- the application directory is `/app` and the server assembly is `/app/SapInvoiceMcpServer.dll`;
- the image entry point serves HTTP, so an administrative verb is reached by overriding it; and
- the manifest command is exactly:

```
docker run --rm --entrypoint dotnet <image> /app/SapInvoiceMcpServer.dll config manifest
```

It writes the manifest as JSON to standard output and nothing else there, exits zero on success,
exits non-zero on any unknown or extra argument, and never opens a listening socket. Its output is
byte-identical across runs of the same image.

The manifest's environment-variable form is authoritative. Do not publish a prefix-and-separator
rule for LocalAI to reimplement, and do not let the two disagree.

## Serve one unauthenticated readiness path

This overrides Prompt 8's blanket rejection of unauthenticated remote-HTTP requests, for exactly one
path. Prompt 9's readiness — local configuration and schema state, never SAP availability — is what
it reports.

- The path is `/health`. `GET` only; every other method is refused.
- It requires no bearer token in either authentication mode, and returns no `WWW-Authenticate`
  challenge, no OAuth metadata and no CORS allowance.
- The body contains the literal token `ready` when the server is ready and does not contain it
  otherwise, so a probe can match one string without parsing.
- It discloses nothing else: no configuration value, secret, path, SAP origin, upstream class,
  version, commit, claim, tenant or exception detail.
- It is still subject to host validation, so a probe has to send one of the allowed Host values
  below. A probe that does not is refused, and that refusal is correct rather than a bug to work
  around by widening host validation.
- Prompt 12F's fake upstream serves the same path for the same reason.

No other unauthenticated path is added. `/mcp` remains authenticated in both modes, and the OAuth
metadata paths from Prompt 12 remain exactly as that prompt defined them.

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

## Fix the advertised OAuth scope for this deployment

Prompt 12 made the advertised scope optional configuration and deferred the deployment's value to
this stage. The value is `sap.invoice.schema.read`.

It is the least-privileged of the four role names, and advertising it is honest about what a
conversational agent must ask for and misleading about nothing: the shared realm mints a constant
`roles` claim, so requesting this scope does not narrow what the resulting token can reach. Say that
where an operator meets the scope, rather than implying the scope is a permission boundary.

The name has to exist in the shared realm as a client scope or Keycloak refuses the authorization
request with `invalid_scope` before a login page appears. LocalAI's shared-host script derives one
realm-default client scope per entry in its role list, so adding the four SAP roles in Prompt 13L
creates this scope as a side effect. Prompt 13L proves that rather than assuming it.

## Make LocalAI the sole development deployment owner

Do not add a Compose project, Caddy template, Windows deployment script or LocalStack client to this
repository. If an earlier server-owned development deployment exists, retire it and mark every
retained historical reference superseded and inert.

Document that LocalAI's `prompts/13L-add-sap-mcp-deployment-to-localai.md` creates and governs
`docs/setup-sap-mcp-windows.ps1`. The server repository owns:

- its container image, the `SOURCE_REVISION` argument and the OCI revision label;
- the Prompt 12F fake-upstream image, on the same terms;
- its configuration manifest, the manifest command line and startup validation;
- transport, host, readiness, authentication and authorization behavior; and
- documentation of what a conforming deployment must supply.

LocalAI owns Git-ref resolution, exact-commit building, Compose, Caddy, host-side secret
materialisation, deployment state, health gating, upgrade, rollback, stop, removal and diagnostics.

## Require explicit deployment inputs

The deployment contract must identify, without inventing values:

- SAP origin, upstream class and deployment profile;
- optional SAP client number;
- SAP credential mechanism and the `File`-sourced path for every required secret;
- allowed company codes and the posting amount/currency policy;
- the accepted tenant, which Prompt 12 requires in both authentication modes;
- fixed-token file path and subject when fixed-token mode is selected;
- Keycloak issuer, audience, resource URI and the advertised scope in Keycloak mode;
- every plan, cursor or signing secret introduced by earlier stages, and an explicit statement that
  there are none if that is the case, so a deployment does not go looking for one;
- exact allowed hosts, CORS origins, limits and timeouts;
- the container alias and the `/health` readiness path; and
- whether a Prompt 12F fake-upstream container is deployed alongside, and if so its origin on the
  private network the two containers share.

No SAP origin, company code, amount, currency, client number, credential path, tenant, profile or
advertised scope receives a plausible default. File secrets stay behind the Prompt 11 `File` source;
no LocalStack, AWS SDK or other vendor secret client enters application code.

State each required value once, in the delivered documentation, in the manifest's canonical naming.
A deployment that has to read this prose to learn a setting name is a contract failure; the prose
exists to say which values a human must decide, and the manifest to say what they are called.

## Prove

- profile and upstream class are both required with no default, and every invalid class, scheme,
  address, profile and secret-source pairing in the table fails startup with its own message;
- specifically: a publicly routable `Fake`, a loopback `Sandbox`, an HTTP `Sandbox`, a private
  `Production` and a `Production` class under a `NonProduction` profile each fail distinctly;
- LocalAI's supported class set cannot express `Production`;
- fake and sandbox responses, including successful post results, carry their class marker;
- the administrative manifest is deterministic, value-free, derived from the bound options,
  available offline and labelled with the image source revision;
- the exact documented manifest command line runs against the built image, writes only JSON to
  standard output, exits zero, opens no socket, and exits non-zero on an extra argument;
- an image built with `SOURCE_REVISION` set reports that value both in the OCI revision label and in
  the manifest, and the two agree;
- every setting the manifest classifies as secret material is unsatisfiable from an environment
  literal, and its environment-variable form is what a deployment actually has to set;
- `/health` answers `GET` unauthenticated in both modes, contains `ready` only when ready, is
  refused for other methods, is still host-validated, and leaks no configuration or secret;
- no unauthenticated path other than `/health` and the Prompt 12 metadata paths exists;
- the trusted-proxy posture requires and accepts the HTTPS external URI while Kestrel remains
  internal HTTP;
- host validation accepts exactly the three configured names, including the healthcheck alias;
- both authentication modes work under the topology and retain their distinct challenges;
- no server-owned deployment entry point or vendor secret provider is introduced; and
- the complete four-tool, one-resource surface and two fixed SAP services remain unchanged.

## Acceptance criteria

- The server image exposes a self-describing configuration contract that LocalAI can validate before
  replacing a running container, reachable by one documented command line.
- Build-argument, label, assembly-path and command-line names are fixed as a cross-repository
  contract rather than left for LocalAI to discover.
- The documented deployment is public HTTPS through shared Caddy with no direct backend listener.
- One unauthenticated readiness path exists, and the rule it overrides is recorded as superseded.
- The LocalAI development deployment is structurally unable to select production SAP.
- Every environment-specific and financially material value is explicit rather than guessed.
- Prompts 12F and 13L are identified as the fake-upstream and deployment implementation stages.
- Formatting, build, schema checks, both container builds and tests succeed.

Commit locally. Use `narrative-required` and record the three superseded rules and why each was
replaced rather than reinterpreted, the profile and upstream-class declaration, non-production
marking, the fixed image and manifest command contract, the single unauthenticated readiness path,
the chosen advertised scope, the final HTTPS trusted-proxy topology and the transfer of deployment
ownership to LocalAI. Do not push unless requested and do not deploy to the host.
