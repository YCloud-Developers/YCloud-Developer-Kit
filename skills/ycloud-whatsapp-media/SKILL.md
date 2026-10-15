---
name: ycloud-whatsapp-media
description: Execute, design, implement, or evaluate the YCloud WhatsApp media upload contract, multipart request and response handling, and media-to-message handoff. Use for media upload integration; do not use for sending messages, template lifecycle, readiness, broad integration planning.
---

# YCloud WhatsApp Media

Upload WhatsApp media or design, implement, and evaluate its integration.
Keep this skill limited to media upload and the handoff that follows it.

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

## Trigger boundary

Use this skill when the user asks to upload WhatsApp media, construct the upload
request, interpret its response, or connect an upload result to a later message.
Route these requests elsewhere:

- Send a message, including a message that references an existing media object: hand off to `ycloud-whatsapp-messages`.
- Create, edit, list, retrieve, delete, or analyze a template: hand off to `ycloud-whatsapp-templates`.
- Design a multi-domain YCloud integration: hand off to `ycloud-integration-architect`.
- Check Developer Kit readiness or run repository-maintenance/issue-tracker workflows: do not trigger this skill.
- For an upload request, inspect and transmit only the user-selected file after validating type/size. A mock-only request uses a synthetic fixture.

Do not load the complete OpenAPI document. Read only
`references/openapi.md` and `references/runtime.md` after this skill has been selected. If retry, idempotency, queueing, or error translation is requested, also read `references/shared/integration-boundaries.md`. Treat the exact
source path, method, operationId, schema, and constraints in that generated
reference as authoritative. If the reference is missing, stale, or conflicts
with the pinned source, report the drift and stop rather than guessing.

## Workflow

1. Resolve the upload target, selected file, and configured connection from
   context. Ask only for missing facts needed to perform that upload. For code
   work, inspect the scoped project runtime and preserve its existing patterns
   and concurrent edits; a direct upload does not require a project or framework.
2. Select the single upload operation documented in the reference:
   `POST /whatsapp/media/{phoneNumber}/upload`, operationId
   `whatsapp_media-upload`. Preserve its path parameter, multipart content type,
   required fields, request schema, response schema, and documented descriptions
   exactly as generated. Apply the standard error envelope, request ID, generic
   `429` headers, documented `413 CONTENT_TOO_LARGE`, the one-file rule,
   reviewed media type/size limits, and 30-day persistence from `runtime.md`.
   Reject multiple files locally even though the API would process only the
   first. Do not invent a media-specific quota, safe replay rule, idempotency,
   or durable retention beyond that period.
   Interpret `allOf` as schema composition and `x-*` extensions or generated
   model names as codegen hints, not additional business behavior.
3. Produce a contract-aware request construction using placeholders such as
   `<YCLOUD_API_KEY>`, `<PHONE_NUMBER>`, and synthetic media metadata. Raw HTTP
   examples are allowed. Use an SDK-specific example only when the project or
   user supplies a confirmed artifact and version; never derive a method name
   from `operationId`.
4. Explain the response only to the extent confirmed by the reference. Identify
   the media identifier/reference needed by a later message, without implying
   that an upload sent a WhatsApp message. Prefer the returned media ID for a
   normal media message, but preserve the documented exception that an
   interactive-message header must use a link instead of a Media ID.
5. Place the upload at the server-side integration boundary (for example, an
   application service or adapter) and keep the API key out of browsers, mobile
   apps, URLs, logs, source control, and generated snippets. Load the configured
   credential only inside the trusted runtime and read only the selected media.
6. Treat upload timeouts as ambiguous and never replay automatically. Any
   project idempotency record, queue, retry budget, or Problem Details response
   is local policy, not a YCloud feature.
7. Give synthetic tests for multipart construction, exactly-one-file validation,
   supported type/size boundaries, 30-day lifecycle handling, path-parameter
   handling, response/reference mapping, the interactive-header link exception,
   and the upload-to-message handoff. These unit tests must not call YCloud or upload a real
   file. Record any authorized live upload separately.

## Outcome requirements

Adapt the result to planning, implementation, execution, or evaluation. Preserve these
contract and evidence outcomes:

1. **Matched contract** — reference source hash, path, method, operationId,
   request content type/schema, response schema, and confirmed constraints.
2. **Request construction** — placeholder raw HTTP or project-local snippet;
   state any project facts still needed.
3. **Response and handoff** — map only confirmed response fields to a media
   reference, then hand off message composition/sending to
   `ycloud-whatsapp-messages`.
4. **Integration placement** — server-side module, configuration boundary, and
   redacted observability guidance.
5. **Tests** — synthetic unit/contract tests and negative cases.
6. **CANNOT** — list unknown contract facts, missing project facts, unsupported
   media operations, SDK uncertainties, and every action intentionally not run.
7. **Handoff** — return to Architect with `whatsapp_media-upload` status,
   changed or proposed artifacts, tests/results, unknowns, and the confirmed
   media-reference contract for Messages; otherwise state that no handoff is
   needed.

## Execution safeguards

A real upload request permits using the selected file and configured credential
within the requested account/phone scope. Verify the returned media reference;
an upload does not prove a message was sent. Do not replace a requested upload
with a synthetic fixture, or upload additional files without authorization.
