---
name: ycloud-api-authentication
description: Execute, design, implement, or evaluate secure server-side YCloud API authentication with the contract’s X-API-Key scheme, placeholder configuration, environment isolation, secret-manager boundaries, and API-key rotation planning. Use for API-key storage, header, or API-key rotation questions; webhook endpoint secret rotation belongs to Webhooks.
---

# YCloud API Authentication

Design, implement, operate, or evaluate authentication at a trusted server
boundary. Use the global scheme in `references/openapi.md` and official error
behavior in `references/runtime.md`. Keep credentials out of visible output.

## Execution boundary

Execute the requested design, local implementation, or real operation within
the user's current request and existing authorization. A clear request to query,
create, edit, delete, upload, send, or rotate authorizes that scoped operation
and its necessary prerequisites. Carry authorization across steps and domain
handoffs; do not ask again merely because an API call or mutation is involved.

Read [the shared execution policy](references/shared/integration-boundaries.md#execution-and-authorization)
before acting. Resolve the account, environment, targets, and operation from
available context; ask only for missing material facts, uncovered effects, or a
required host approval. Use configured credentials through a trusted runtime
without displaying secrets. Local development alone does not authorize unrelated
live actions; keep explicitly requested mock/sandbox work isolated.

For live work, validate the selected contract, execute through an available tool
or server-side client, and verify the result. Report actual status, request IDs,
and unresolved outcomes; distinguish mock evidence, acceptance, and final state.
Honor scoped Architect handoffs when present, but do not require a handoff or a
local project for a direct API task. Reuse the existing project stack for code
changes and preserve concurrent work. For local implementation, use synthetic
fixtures and no-network tests.

## Scope and handoff

Handle server-side API-key configuration, authorized credential use and validation, and rotation through a supported configured mechanism. Do not invent a key-management API. Resolve a generic "rotate secret" request from context; ask only if the target remains ambiguous: webhook endpoint secret rotation belongs to Webhooks, not Authentication. Route these requests away:

- Message send/retrieve (including sending an existing template) → `ycloud-whatsapp-messages`.
- Media upload → `ycloud-whatsapp-media`.
- Template lifecycle or analytics → `ycloud-whatsapp-templates`.
- Webhook endpoint management → `ycloud-webhook-endpoints`.
- Broad or multi-domain integration planning → `ycloud-integration-architect`.
- Plugin readiness → the readiness-only smoke Skill; repository-maintenance and issue-tracker workflows are outside this Plugin.

## Workflow

For explicitly requested sandbox/mock/no-real-side-effect tests, also read
`references/shared/sandbox-contract.md`. The fixed synthetic key belongs only to
the local Facade and is not a YCloud test credential or an example of production
key format.

1. Resolve the configured connection and authorized environment from context. For code/configuration work, inspect the scoped project runtime, configuration mechanism, and test harness; a direct credential validation does not require project files. For authorized operations, load only the configured credential into the trusted runtime without displaying its value; do not search unrelated secret stores, logs, shell history, or browser storage. Ask only for a missing runtime/deployment fact that changes the guidance.
2. Read both narrow references and preserve their exact boundaries. The request header is `X-API-Key`; show only a placeholder such as `<YCLOUD_API_KEY>`, never a user-provided key. An invalid key is documented as HTTP `401` with `error.code=UNAUTHORIZED`; preserve that provider envelope and `YCloud-Request-ID`. If the project exposes RFC 9457, translate only at its own API boundary and retain redacted `provider_code`/`provider_request_id`; never claim YCloud returned Problem Details.
3. Keep the key on a trusted server boundary. Recommend an environment-specific secret injection path or Secret Manager reference, least-privilege access, redaction, and rotation ownership without claiming provider-specific behavior that the contract does not state.
4. Explain environment isolation (development, staging, production), startup/configuration validation that checks presence and shape without printing the value, and tests that use a synthetic placeholder. For local implementation, use the project's established configuration pattern and no-network tests. For a requested live validation, use a scoped documented read operation and report status and request ID without the key.

## Safe examples

Illustrative raw HTTP only; do not execute it:

```http
X-API-Key: <YCLOUD_API_KEY>
```

Illustrative server configuration shape (choose the project’s established mechanism; the value is never supplied here):

```text
YCLOUD_API_KEY=<injected-secret-placeholder>
```

Never place the key in a browser/mobile bundle, client-side storage, a URL/query parameter, source control, a code sample with a real value, analytics, or ordinary request logs. Redact authorization headers in diagnostics.

## Outcome requirements

Adapt the shape to planning, implementation, execution, or evaluation. Make these results
easy to verify:

### Contract

Name the confirmed global `api_key` `apiKey` security scheme with header name `X-API-Key` and cite the generated reference. Distinguish raw HTTP from any SDK/codegen shape.

### Configuration Plan

Describe the server-only injection point, environment separation, access/redaction controls, startup checks, and rotation handoff. Use names and placeholders, not secret values.

### Project Integration

Map the plan to confirmed project modules (HTTP client, config loader, deployment manifest, and tests). If the project cannot be inspected, ask the minimum questions or mark the assumption.

### Tests

Cover header construction with `<YCLOUD_API_KEY>`, missing/blank configuration, environment isolation, redaction, a synthetic `401/UNAUTHORIZED` envelope, and request-ID correlation. These unit tests must not contact YCloud or load a real secret. Report authorized live checks separately.

### CANNOT

Do not expose a key client-side, in a URL, logs, source control, or chat; access unrelated credentials; perform out-of-scope deployment changes; infer an SDK auth method; or assert unspecified key expiry, rotation overlap, recovery, retry, or endpoint-specific rate behavior. Do not mark documented `401/UNAUTHORIZED` behavior as unknown. If a requested key rotation has no supported mechanism, report that limitation; otherwise use the existing authorization and verify the cutover without exposing either key.

### Handoff

Send message/media/template/webhook work to its domain Skill. For an Architect
handoff, return `crosscutting:api-authentication` status, project seams, changed
or proposed artifacts, tests and results, unknowns, and the server-side header
boundary. Authentication does not perform the downstream workflow.

## Codegen boundary

Do not infer a Java, TypeScript, Python, Go, or PHP SDK method from `operationId` or model names. Provide a raw HTTP/header example unless the user supplies a confirmed SDK artifact/version and authoritative docs. Treat generated models and schema composition as codegen hints, not additional authentication behavior.
