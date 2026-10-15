---
name: ycloud-whatsapp-templates
description: Execute, design, implement, or evaluate YCloud WhatsApp template create, list, retrieve, edit, delete, and analytics operations. Use for template lifecycle integration; do not use for sending template messages, media uploads, readiness, broad integration planning.
---

# YCloud WhatsApp Templates

Create, query, edit, delete, and analyze WhatsApp templates, or design,
implement, and evaluate their lifecycle integration.

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

Use this skill for template creation, listing, retrieval, editing, deletion, or
analytics. A request to send a message that happens to use a template belongs to
`ycloud-whatsapp-messages`, even when the template already exists. A request to
upload media belongs to `ycloud-whatsapp-media`. Broad integration design belongs
to `ycloud-integration-architect`; readiness, repository-maintenance, and issue-tracker prompts do
not trigger this skill.

Read only `references/openapi.md` and `references/runtime.md` after selection.
For create requests, component editors, or template smoke tests, also read
`references/create-examples.json`. If retry, idempotency, queueing, or error
translation is requested, also read `references/shared/integration-boundaries.md`. The generated OpenAPI reference is
the contract index for the seven allowlisted template operations: create, list,
retrieve, edit, delete-by-name, delete-by-name-and-language, and analytics. Use
the exact source-derived path, method, operationId, parameter, schema, response,
and descriptions from the reference; do not guess names or paths from prose.
For template-list work, also read `references/shared/pagination-contract.md`.
Interpret `allOf` as schema composition and `x-*` extensions or generated model
names as codegen hints, not additional lifecycle behavior.
If source hash, operation coverage, or reference content drifts, report the drift
and stop.

## Workflow

1. Establish the user’s intended lifecycle operation and the target server-side
   context. Resolve account/environment and template identity from available
   evidence; ask only for missing material facts. A direct API task does not
   require choosing a language or framework.
2. Match exactly one of the seven operations in the generated reference. Keep
   lifecycle operations separate from message sending. Preserve source-defined
   path parameters, request/response schemas, enum values, and description-only
   constraints. Use the runtime reference for the shared Management API quota,
   standard error envelope, request ID, pagination behavior, edit replacement
   semantics, extensible status handling, and documented template error codes.
   Do not fill gaps with imagined safe replay, idempotency, approval states,
   delivery semantics, or analytics meanings.
3. Generate raw HTTP or contract-aware typed request examples, or implement
   project-local request construction when authorized, with placeholders
   such as `<YCLOUD_API_KEY>`, `<TEMPLATE_NAME>`, `<LANGUAGE>`, and synthetic
   values. Use an SDK-specific method only when a confirmed SDK artifact/version
   and documentation are present in the user’s project; `operationId` is not an
   SDK method name.
   For the default create smoke test, use the documented
   `utility-order-confirmation` fixture unchanged except for replacing
   `<WABA_ID>` and, when collision avoidance is required, the template name.
   Use the documented `authentication-copy-code` fixture for authentication
   shape tests. `AUTHENTICATION`, `MARKETING`, and `UTILITY` are top-level
   category values, never component types. Do not manufacture a kitchen-sink
   request by adding one instance of every component enum; component
   combinations have cross-field constraints and must come from a selected
   documented example or a user-supplied valid design.
   Keep `wabaId` as part of template identity across list, retrieve, and edit.
   A UI or local route may encode a composite key, but its provider adapter must
   preserve the exact `/whatsapp/templates/{wabaId}/{name}/{language}` path for
   retrieve and edit; `name|language` alone is not a provider identity.
   Keep template definition components separate from message-send parameters:
   Generate template definitions using the canonical uppercase `CAROUSEL`,
   `HEADER`, `BODY`, and `BUTTONS` spellings (including nested `buttons`).
   Preserve case-insensitive parsing of definition component types, formats,
   and button types; capitalization alone does not distinguish a definition
   from a send payload. Route message parameter composition to
   `ycloud-whatsapp-messages`, which generates the lowercase send component model.
   For carousel create/edit fixtures, include 2..10 cards, a media `HEADER`
   and a `BUTTONS` component containing 1..2 buttons per card. Supply media
   examples as a non-empty `example.header_url` string array. The service uses
   its first URL, which must be HTTP(S) with `.jpg`, `.jpeg`, or `.png` for
   `IMAGE` and `.mp4` for `VIDEO`. If a card
   includes `BODY`, its text must be non-blank and at most 160 characters.
   The two-card minimum is a definition constraint, not a minimum count for
   the parameter overrides included in a later send request.
   Apply the same request validator before both sandbox storage and live
   transport so live mode cannot bypass component/category/button validation.
4. Keep API keys server-side and out of browser/mobile code, URLs, logs, source
   control, and snippets. Use scoped credential injection for live operations
   and read only templates or analytics needed for the request.
5. For create/edit/delete, describe the planned change and its validation and
   rollback considerations. An edit is a full content replacement: include all
   components that must survive, and allow it only for `APPROVED`, `REJECTED`,
   or `PAUSED`; never edit `ARCHIVED`. For list, use 1-based `page`, `limit`
   1..100, and `includeTotal` only when a count is required; archived matches
   remain visible. Parse the response as the merged Page envelope with required
   `offset`, `limit`, `length`, template `items`, and optional `total`; `offset`
   is response metadata, not a query parameter. Preserve unfamiliar status values and stop state-dependent
   mutation rather than mapping them to a known state.
6. Build synthetic tests for request/schema validation, full-replacement edits,
   editable/archived/unknown status gates, pagination boundaries and optional
   totals, path/query parameters, response mapping, and the selected lifecycle
   branch. Create tests must assert that category values never appear in
   `components[*].type` and that the selected documented example survives
   request construction without extra components. Retrieve/edit tests must
   assert that the captured provider path includes the WABA ID. Also test that
   live and sandbox handlers reject the same invalid component payload before
   transport. These unit/contract tests use mocks; report live checks separately.
7. Treat create/edit/delete timeouts as ambiguous outcomes and never replay
   automatically. Any project idempotency ledger, outbox, retry budget, or RFC
   9457 response is local architecture, not a YCloud contract.

## High-risk delete handling

For both delete operations, establish the WABA, name, and whether deletion
affects one language or all matching languages. If that scope and effect are
already authorized, execute without asking again; otherwise clarify only the
missing boundary. Verify the outcome and describe rollback only when supported. Never imply that deleting a template withdraws or changes
already-submitted messages.

## Outcome requirements

Adapt the result to planning, implementation, execution, or evaluation. Preserve these
contract and evidence outcomes:

1. **Matched contract** — source hash, selected operation, exact path/method,
   parameters, request/response schemas, and confirmed constraints.
2. **Lifecycle plan** — placeholder raw HTTP or project-local construction and
   the intended create/list/retrieve/edit/delete/analytics behavior.
3. **Safety and integration placement** — server-side boundary, credential
   handling, validation, and exact deletion scope under existing authorization.
4. **Tests** — synthetic contract, mapping, and negative tests with no live call.
5. **CANNOT** — unknown metrics or runtime behavior, unsupported operations,
   unconfirmed SDK methods, missing project facts, and actions not performed.
6. **Handoff** — route template message composition/sending to
   `ycloud-whatsapp-messages`; return to Architect with each selected operation's
   row status, changed or proposed artifacts, tests/results, lifecycle/status
   preconditions, unknowns, and outgoing handoffs.

## Non-goals

Route webhook consumption, signature verification, media upload, and message
sending to their owner Skills when required by the authorized task. Do not make
unrequested project changes or broaden the operation scope. A
template lifecycle result is not a sent or delivered message; keep that
distinction in every example and handoff.
