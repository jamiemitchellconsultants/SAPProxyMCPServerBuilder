# Prompt 12 — Use the shared home-lab Keycloak OAuth flow

Using the reusable contract and the artifacts produced by Prompts 1–11, make the JWT branch usable
by tier-one conversational agents through the shared home-lab Keycloak realm.

This stage changes JWT configuration and OAuth discovery only. It must not widen the MCP surface,
alter SAP service authentication, make authorization claims optional, or weaken company-code,
amount, supplier-visibility, plan, confirmation or no-retry controls. Prompt 11's fixed-token mode
remains fully supported and selectable.

## Use the shared realm and audience

The home-lab deployment uses:

- issuer `https://auth.tqaentry.com/realms/homelab`;
- audience `homelab-mcp`, shared by every MCP server in the lab;
- a mandatory flat `tid` claim; and
- a mandatory flat `roles` array containing the role needed by the capability being exercised.

Keep issuer and audience as reviewed configuration rather than hard-coded application constants.
Locally validate signature, pinned algorithms, issuer, audience, expiry, not-before and required
claims. Continue deriving caller and tenant only from validated claims.

## Map roles to permissions by identity

The realm mints one flat `roles` array. Its values are Prompt 8's permission names, exactly, with no
transformation:

- `sap.invoice.plan`
- `sap.invoice.post`
- `sap.invoice.payment.read`
- `sap.invoice.schema.read`

A role grants the identically named permission and nothing else. Do not introduce a prefix, suffix,
case change, alias table, scope-to-role mapping or role hierarchy, and do not let an unrecognised
role value grant anything. These are the same four permission names Prompt 11 configures on the
fixed-token identity, so both modes present identically shaped input to the one deny-by-default
authorization service.

This identity is the single fact three repositories have to agree on. State it in the server's
delivered documentation as a contract, not as an implementation detail.

## Configure the accepted tenant in both modes

The shared realm mints a constant `tid`. A validated token is not evidence that its tenant is one
this deployment serves. Add one required configured accepted-tenant value:

- in Keycloak mode, a token whose validated `tid` is any other value is refused after signature and
  claim validation and before authorization; and
- in fixed-token mode, the same setting is the static identity's tenant from Prompt 11.

The setting is required in both modes. There is no default, no wildcard and no "any tenant". An
absent value fails startup by name.

Document that this is a development-lab decision: a token minted for another MCP server can pass the
audience check here. The audience is not evidence of which server the caller intended to reach. The
required roles and the existing business-policy checks remain the authorization boundary.

The lab supplies the same tenant and SAP roles to every authenticated user. Do not introduce a
claims-optional or authorization-disabled path to represent that. Production identity remains
outside this stage and must not be described as using this shared realm.

## Serve OAuth protected-resource metadata where clients look

In the JWT branch only, serve one deterministic metadata document at both unauthenticated paths:

- `/.well-known/oauth-protected-resource`; and
- `/.well-known/oauth-protected-resource/mcp`.

The resource URI is the configured external HTTPS `/mcp` URI. The authorization-server entry names
the configured Keycloak issuer. An unauthenticated MCP request returns a Bearer challenge whose
`resource_metadata` value is derived from that configured resource URI.

Make the advertised scope optional:

- when configured, it is one non-blank token, appears in metadata and the challenge, and is
  documented as requiring a matching Keycloak client scope; and
- when absent, configuration remains complete, metadata omits `scopes_supported`, and the challenge
  omits `scope`.

Do not advertise a scope that the shared realm has not created, and do not choose the deployment's
value here. Prompt 13 fixes it, because it owns the deployment contract; Prompt 13L consumes that
value and proves the matching client scope exists as a realm default.

In fixed-token mode, preserve Prompt 11 exactly: neither metadata path is served and the challenge
is a bare Bearer challenge with no metadata or scope.

## Support dynamically registered conversational clients

The server remains a resource server. It does not gain an authorization endpoint, token endpoint,
registration endpoint, session store, browser flow or token introspection call. Keycloak owns those
functions.

A conversational agent may register a new public client and use authorization-code with PKCE. Its
client identifier was not reviewed by this repository. Verify that `azp`, `appid`, client ID and
redirect URI are never treated as authorization evidence. Device flow may remain a fallback for
non-agent tooling, but this server neither runs nor emulates it.

## Preserve both exclusive authentication modes

Rename the former generic JWT mode to `Keycloak` if that makes the configuration contract explicit;
otherwise document the exact existing name. In either case there are still exactly two exclusive
modes: Keycloak and fixed token. Neither falls back to the other, and selecting a mode constructs
only its authentication branch.

Both modes can reach the complete four-tool, one-resource surface only through the same
deny-by-default authorization and business-policy services.

## Prove

- both metadata paths return the same unauthenticated document in Keycloak mode and neither is
  served in fixed-token mode;
- configured scope is advertised and challenged, while absent scope is omitted and passes startup
  validation;
- Keycloak mode challenges with metadata derived from the configured HTTPS resource URI, while
  fixed-token mode returns the bare challenge;
- a token carrying `homelab-mcp`, the configured issuer, tenant and required role validates, while
  missing or wrong values are refused;
- each of the four role values grants exactly its identically named permission, an unrecognised role
  value grants nothing, and a token carrying every role still fails the business-policy checks it
  should fail;
- a validated token whose `tid` is not the configured accepted tenant is refused before
  authorization, and an absent accepted-tenant setting fails startup in both modes;
- signature, algorithm, expiry, not-before and claim failures remain fail closed;
- no client identifier or dynamic-registration fact grants authority;
- both authentication modes reach the same surface and neither falls back; and
- denial still occurs before SAP secret-provider or network access.

## Acceptance criteria

- A conversational agent given only the external `/mcp` URI can discover Keycloak, register itself,
  complete authorization-code with PKCE and call an authorized read-only tool.
- Fixed-token mode remains unchanged and serves no OAuth metadata.
- Role values and permission names are identical, documented as a cross-repository contract.
- One accepted tenant is required configuration in both authentication modes.
- The shared-audience and constant-claim costs are documented without weakening authorization.
- The MCP surface and SAP credential contract are unchanged.
- Formatting, build, schema checks and tests succeed.

Commit locally. Use `narrative-required` and record the shared audience and its cost, the identical
role-and-permission naming and why it is a contract rather than a coincidence, why one accepted
tenant is required in both modes, why both metadata paths are served, why advertised scope is
optional, and why dynamically registered client identity is not authorization evidence. Do not push
unless requested.
