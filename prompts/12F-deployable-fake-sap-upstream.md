# Prompt 12F — Package the fake SAP upstream as a deployable container

Using the reusable contract and the artifacts produced by Prompts 1–12, package the existing fake
SAP server as a separately deployable container image so a home-lab deployment can run the server
against it across a container network.

Play this stage before Prompt 13. Prompt 13 declares a `Fake` upstream class reachable at a
container-network address, and Prompt 13L lets the LocalAI deployment select a fake upstream. Prompt
3's fake is a loopback test double inside the test host process; nothing in Prompts 1–12 produces an
image, so without this stage `Fake` names an upstream that does not exist.

This stage adds no capability to the SAP MCP server and does not touch its image, MCP surface,
authentication, authorization or SAP client. It packages a test double.

## Extract the fake once, and use it in both places

Move the Prompt 3 fake into one project — `tools/SapInvoiceMcp.FakeUpstream` or the equivalent name
already in use — referenced by the test project and built into its own image. There is exactly one
fake implementation. Do not fork a second copy for the container, and do not let the container form
diverge by a single route, field or error.

The served surface is exactly Prompt 3's and does not grow:

- `$metadata` for the two fixed services;
- `POST .../API_SUPPLIERINVOICE_PROCESS_SRV/A_SupplierInvoice`;
- CSRF token acquisition for that service; and
- bounded `GET` on the operational-accounting item entity.

Every other path, method, entity, query option and service root returns a flat rejection that
discloses no route list, no entity model and no framework diagnostic. Adding a route to serve the
container is adding it to the test double, and Prompt 3's exact-path assertions must still fail if
anything else answers.

## Make it structurally unable to reach SAP

The fake serves synthetic data and has no upstream. It contains no outbound HTTP client, no
configurable origin, proxy or forwarding target, and no credential of any kind. There is nothing an
operator can set that causes it to relay a request, and nothing a caller can send that causes it to
resolve a hostname. State this as the reason it is safe to run on a network the SAP server can
reach.

Synthetic invoices, suppliers, purchase orders, payments, clearing evidence and errors come from a
deterministic seed selected in configuration. State is in memory only: it does not persist across a
restart, and the deployment documentation says so rather than implying a durable sandbox.

## Bind for a container network

This section overrides Prompt 3's loopback binding for the containerised form only. The in-process
test double keeps binding loopback; nothing about the test topology changes.

- The image listens on internal HTTP `0.0.0.0:<port>`, with the port from configuration.
- TLS is not terminated in the fake and is not required of it. Prompt 13's `Fake` class is the rule
  that permits plain HTTP, and it permits it only on a loopback, private or container-network
  address.
- The fake performs no caller authentication and says so plainly. It is reachable by anything on
  the network it joins, which is why Prompt 13L keeps it off the shared public ingress network and
  publishes no host port for it.
- It answers unauthenticated `GET /health`, whose body contains the literal token `ready` when it is
  ready and does not otherwise. Prompt 13 fixes the same path and token for the server, so one
  health-probe shape works for both containers.

## Build it like the server, separately from the server

- Its own multi-stage build producing a runtime-only image with no SDK tooling, and its own pinned
  runtime base.
- A `SOURCE_REVISION` build argument defaulting to `unknown`, recorded in the image as OCI label
  `org.opencontainers.image.revision`. Prompt 13 fixes the same two names for the server image, so a
  deployment can prove both containers came from one commit by reading one label from each.
- A distinct image title and description that name it a fake. An operator reading `docker image ls`
  or a label dump must not be able to mistake it for the SAP MCP server or for SAP.
- Non-root, no writable application directory, no published host port in any example.
- It is never included in, layered onto or reachable from the server image. The server image gains
  no fake, no test fixture and no seed data.

## Prove

- one implementation serves both the test double and the image, and a route added for one appears
  in the other's exact-path assertions;
- the container serves exactly the four Prompt 3 behaviours and rejects every other path, method,
  entity and query option without disclosure;
- no outbound HTTP client, upstream origin, proxy setting or credential exists anywhere in the
  project, proven from source rather than from configuration defaults;
- a fixed seed produces identical responses across restarts, and state does not survive one;
- the image listens on `0.0.0.0` at the configured port, serves the fixed readiness path
  unauthenticated, and publishes no host port in any documented example;
- image labels report the build commit and identify the image as a fake; and
- the server image contains no part of it, and the four-tool, one-resource MCP surface, both SAP
  services and both authentication modes are unchanged.

## Acceptance criteria

- A `Fake` upstream exists as a deployable image built from the same commit as the server.
- It is one implementation shared with the tests, not a second fake that can drift.
- It cannot be configured to contact a real SAP system, and cannot be mistaken for one.
- Its plain-HTTP, unauthenticated, non-durable nature is documented rather than softened.
- Formatting, build, schema checks, both container builds and tests succeed.

Commit locally. Use `narrative-required` and record why the fake became a deployable artifact, why
it stayed a single implementation shared with the tests, why it carries no outbound client at all
rather than a disabled one, and that it is never part of the server image. Do not push unless
requested.
