---
date: 2026-08-05
slug: add-ai-mcp-server-deployment-prompts
title: "Add ai-mcp-server deployment prompts"
summary: "Extend the sequence with fixed-token authentication, shared home-lab Keycloak OAuth, a deployable fake upstream, a final HTTPS trusted-proxy deployment contract and a cross-repository audit."
kind: product
status: accepted
sequence: 2026-08-05T14:51:17.000Z
evidence: "https://github.com/jamiemitchellconsultants/SAPProxyMCPServerBuilder/pull/6; merge commit 0269bcc97e7bcf8e3a4e6661bd263ff6cadf9d26"
---

## Context

The builder ended with a packaged, authenticated SAP supplier-invoice server but did not teach how to deploy it to the existing `ai-mcp-server` home-lab host. That host's shared Caddy, Keycloak, LocalStack and `mcp-public` network are owned by LocalAI, while the server must retain ownership of its closed MCP surface, SAP boundaries, transport, authentication and configuration validation.

A review of the first draft of these stages against Prompts 1-10 found that the two halves could not meet. Prompt 11 read the fixed token through a "file secret-provider boundary" that Prompt 3 never established. Prompt 13 declared a `Fake` upstream class on a container-network address when Prompt 3's fake is a loopback test double inside the test host process, so nothing built an image. Prompt 13 asserted that every Prompt 3 URL rejection still held while publishing a table that contradicted one of them, and validated against a deployment profile Prompt 3 only alluded to. The generated topology needed an unauthenticated health probe against Prompt 8's blanket rejection of unauthenticated requests. Neither repository named the manifest command, the server assembly, the build argument or the label — the four strings that must exist before a deployment script can be written at all. The advertised OAuth scope was deferred from Prompt 12 to 13L and declined by 13L, so nobody chose it.

## Decision

Extend the sequence with fixed-token authentication, shared home-lab Keycloak OAuth, a deployable fake upstream, a final HTTPS trusted-proxy deployment contract and a cross-repository audit. Fixed-token authentication is played before OAuth. The deployment stage requires an explicit fake, sandbox or production upstream declaration, limits LocalAI to fake or sandbox, and makes the server image emit a value-free configuration manifest so LocalAI validates the exact candidate and rollback image.

Resolve every dependency gap by overriding in the later prompt rather than editing an earlier one, and require each override to be recorded as a supersession in the prompt that makes it:

- Prompt 11 adds one `File` secret source covering every secret kind Prompt 3 defined, and closes the environment-variable route to any secret-classified setting.
- Prompt 12 states role-to-permission naming as a cross-repository contract rather than a coincidence, and requires one configured accepted tenant in both authentication modes.
- Prompt 12F packages the existing fake as its own image, shared with the tests so it cannot drift, with no outbound HTTP client at all rather than a disabled one.
- Prompt 13 records three superseded rules, enumerates a two-value deployment profile, constrains the upstream-class table in both directions, fixes `/health` as the single unauthenticated path, fixes `SOURCE_REVISION`, the OCI revision label, `/app/SapInvoiceMcpServer.dll` and the manifest command line, and chooses `sap.invoice.schema.read` as the advertised scope.
- Prompt 14 audits the supersessions and the fixed image contract, and fails a LocalAI script that derives an environment-variable name by transformation instead of reading the manifest's declared form.

## Consequences

Completing the deployment now requires coordinated, separately reviewed changes in the server and LocalAI repositories. The home-lab endpoint is public HTTPS behind shared Caddy, uses a shared audience and grants the same configured SAP roles to lab users, while the existing tenant, company-code, amount, supplier, plan and confirmation controls remain mandatory. Production SAP cannot be selected by the LocalAI development deployment, and no prompt application authorises a live SAP write.

The sequence is now sixteen stages rather than fifteen, and three of them weaken a rule an earlier stage established. That is intentional and is the reason each supersession is stated where it happens and re-checked by Prompt 14: a reader who meets only the earlier prompt would otherwise treat the later behaviour as a defect, and a reader who meets only the later one would not know a decision was reversed. `/health` in particular is a real reduction in surface protection, accepted because a health-gated deployment cannot exist without it.

Four names are now a published contract between three repositories. Changing `SOURCE_REVISION`, the OCI revision label, the server assembly path or the manifest verb is a breaking change to the LocalAI deployment, not an internal refactor.
