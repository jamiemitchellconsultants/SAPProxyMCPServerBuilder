# Prompt 11 — Add home-lab fixed-token authentication

Using the reusable contract and the artifacts produced by Prompts 1–10, add a second caller
authentication mode for a physically controlled home-lab development network: one fixed opaque
bearer token selected explicitly at startup.

This stage changes caller authentication only. It must not widen the four-tool, one-resource MCP
surface, change either fixed SAP service, weaken authorization or confirmation, or make a live SAP
write part of an automated test. The JWT mode from Prompt 8 remains supported. Prompt 12 will adapt
that mode to the shared home-lab Keycloak realm and OAuth discovery flow.

## Select exactly one mode

Exactly two remote-HTTP modes exist:

- `Jwt`, preserving Prompt 8's signature, issuer, audience, lifetime and required-claim validation;
  and
- `FixedToken`, comparing one presented opaque bearer token with a value read from a file.

The mode is selected once from reviewed startup configuration. An absent, unknown, ambiguous or
incomplete selection fails startup before the transport accepts a request. Name those four operator
mistakes separately and name the missing setting without ever printing a secret value.

There is no fallback in either direction. Construct only the selected authentication branch. An
unreachable issuer, failed JWKS retrieval or rejected JWT never tries the fixed token. In
fixed-token mode a JWT is merely an opaque string: it is never parsed and never reaches the JWT
validator.

## Implement the fixed-token provider

- Read the expected token through the existing file secret-provider boundary. Do not accept it from
  an environment variable, command argument, configuration literal, repository fixture or Docker
  label.
- Fail startup when the file is absent, unreadable, empty or whitespace-only.
- Reject an absent or malformed `Authorization` header before comparison.
- Compare the presented and expected values in constant time. A near miss must be indistinguishable
  from any other wrong token in timing, response shape and logging.
- Redact the value completely from logs, traces, metrics, audit records, health, errors, exceptions,
  diagnostics and generated configuration.
- Return a bare `WWW-Authenticate: Bearer` challenge. Fixed-token mode serves no OAuth
  protected-resource metadata and advertises no authorization server or scope.

## Establish one configured caller

A valid fixed token establishes one deterministic caller identity from configuration alone:

- one configured subject;
- one configured tenant;
- all four existing SAP permissions; and
- the existing configured company-code, amount and supplier-visibility boundaries.

No identity or authorization field is derived from the token. The token proves possession only.
The authorization service still evaluates every call, including plan execution, and denial still
occurs before SAP credential or network access.

Document the accepted consequences plainly:

- every token holder is the same caller, so audit attribution, rate-limit buckets and caller-bound
  plan scope collapse onto one identity;
- a plan issued to that identity can be used by any token holder, subject to all other plan checks;
- the token has no expiry, per-holder revocation or claim corroboration; rotation means replacing
  the file and restarting; and
- whoever can read the deployment's secret store holds the credential outright.

These consequences are accepted only for this home-lab development mode. Do not add a warning-only
fallback, silently disable invoice posting, or pretend that the network supplies caller identity.

## Prove

- absent, unknown, ambiguous and incomplete mode selections fail startup distinctly and without
  exposing secret material;
- composition constructs exactly one authentication branch;
- a JWT in fixed-token mode never reaches JWT validation, and a valid fixed token in JWT mode is
  refused;
- missing, unreadable, empty and whitespace-only files fail before request acceptance;
- comparison is constant time and secret values appear in no observable output;
- the configured identity can reach all four tools and the ontology resource only when the existing
  permission and business-policy checks allow it;
- every decision is audited against the configured subject and tenant;
- fixed-token 401 responses carry only the bare Bearer challenge and no metadata endpoint is
  served; and
- Prompt 8's JWT validation and denial-before-SAP behavior remain unchanged.

## Acceptance criteria

- Remote HTTP selects exactly one of two fail-closed authentication modes with no fallback.
- Fixed-token mode reads one file-backed secret and creates one configured shared identity.
- The shared-identity, audit, plan-scope, expiry, revocation and secret-store consequences are
  documented rather than disguised.
- The MCP and SAP surfaces are unchanged, and automated tests contact only the fake SAP server.
- Formatting, build, schema checks and tests succeed.

Commit locally. Use `narrative-required` and record the exclusive mode choice, the shared identity
and its consequences, and the decision to retain Prompt 8's JWT path unchanged until Prompt 12.
Do not push unless requested.
