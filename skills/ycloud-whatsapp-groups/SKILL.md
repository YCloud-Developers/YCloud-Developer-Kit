---
name: ycloud-whatsapp-groups
description: Execute, design, implement, or evaluate all 12 YCloud WhatsApp Groups operations, including asynchronous lifecycle tracking, join requests, participants, settings, invite links, and the invite-link message handoff. Use for group management; exclude ordinary or Flow message sending, Flow lifecycle.
---

# YCloud WhatsApp Groups

Manage WhatsApp groups or design, implement, and evaluate their integration.
Distinguish an accepted request from its final asynchronous state.

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

Keep claims in three authority layers:

1. **Provider contract** — [references/openapi.md](references/openapi.md) and
   [references/runtime.md](references/runtime.md). State these as YCloud behavior.
2. **Developer Kit policy** — `references/shared/integration-boundaries.md` when
   retry, idempotency, queueing, error translation, or webhook reliability is in
   scope. Label its recommendations as local policy.
3. **Project decisions** — only facts confirmed in the user's scoped project.

Do not promote an `operationId`, generated model name, `x-*` extension, example,
platform recommendation, or project convention into provider behavior. If the
generated references are absent, stale, internally inconsistent, or do not list
all 12 operations below, stop and report the drift.

## Exact operation allowlist

Load both generated references after this Skill is selected. Match only these
operations and preserve every source-defined path parameter, query parameter,
request/response schema, status code, and description constraint.
For either list operation, also read
[`references/shared/pagination-contract.md`](references/shared/pagination-contract.md).

| Intent | Method and path | operationId |
| --- | --- | --- |
| Create group | `POST /whatsapp/{businessPhoneNumber}/groups` | `whatsapp_group-create` |
| List groups | `GET /whatsapp/{businessPhoneNumber}/groups` | `whatsapp_group-list` |
| Retrieve group | `GET /whatsapp/{businessPhoneNumber}/groups/{groupId}` | `whatsapp_group-retrieve` |
| Delete group | `DELETE /whatsapp/{businessPhoneNumber}/groups/{groupId}` | `whatsapp_group-delete` |
| Retrieve invite link | `GET /whatsapp/{businessPhoneNumber}/groups/{groupId}/inviteLink` | `whatsapp_group-retrieve-invite-link` |
| Reset invite link | `POST /whatsapp/{businessPhoneNumber}/groups/{groupId}/inviteLink/reset` | `whatsapp_group-reset-invite-link` |
| Send invite-link message | `POST /whatsapp/{businessPhoneNumber}/groups/inviteLink/messages` | `whatsapp_group-send-invite-link-message` |
| List join requests | `GET /whatsapp/{businessPhoneNumber}/groups/{groupId}/joinRequests` | `whatsapp_group-list-join-requests` |
| Approve join requests | `POST /whatsapp/{businessPhoneNumber}/groups/{groupId}/joinRequests/approve` | `whatsapp_group-approve-join-requests` |
| Reject join requests | `POST /whatsapp/{businessPhoneNumber}/groups/{groupId}/joinRequests/reject` | `whatsapp_group-reject-join-requests` |
| Update settings | `PATCH /whatsapp/{businessPhoneNumber}/groups/{groupId}/settings` | `whatsapp_group-update-settings` |
| Remove participants | `POST /whatsapp/groups/{groupId}/participants/remove` | `whatsapp_group-remove-participants` |

The participant-removal path intentionally omits `{businessPhoneNumber}`. Do
not normalize it to the shape of the other endpoints. Group conversation message
composition or sending belongs to `ycloud-whatsapp-messages`; only the dedicated
invite-link-message operation remains in this Skill. Flow lifecycle belongs to
`ycloud-whatsapp-flows`.

## Contract-first workflow

1. Confirm the selected operation and server-side project seam. Keep
   `businessPhoneNumber` in source-required E.164 form. Treat `groupId`,
   `joinRequestId`, `requestId`, user/parent-user IDs, message IDs, and cursors as
   opaque, case-sensitive strings; never parse examples, prefixes, suffixes, or
   cursor contents. Preserve unknown response properties and enum/event values.
2. Preserve request invariants exactly. Group create requires `subject`, applies
   source length limits, and defaults omitted `joinApprovalMode` to
   `auto_approve`. Settings updates require at least one of `subject` or
   `description`. Participant removal supports at most eight entries, each with
   exactly one of `user` or `fromUserId`; `userId` is a documented alias for
   `fromUserId`. Join-request actions pass the opaque IDs returned by list and
   retain per-item failures instead of treating a mixed result as atomic.
3. Preserve cursor pagination for group and join-request lists: `limit` is 1 to
   1024 with a default of 25, and `before`/`after` are opaque. Do not invent page
   numbers, totals, stable ordering, or combine both cursor directions unless
   the generated reference explicitly permits it. Parse the provider response as
   `{data: [...], paging?: {before?, after?}}`; it is not the common
   `offset/limit/length/items` Page envelope.
4. For invite-link messages, require an approved template name, language code,
   ordered body parameters, and one `type=group_id` parameter whose `group_id`
   is the target group ID. The message goes to one user, not into the group.
   Exactly one of `to` or `recipient` is required; if both exist, resolve and
   validate `to` first and ignore `recipient` completely. Never fall back to
   `recipient` when `to` is invalid. Hand accepted message state to Messages.
5. Use placeholders and synthetic IDs/data in examples and tests. Use an
   SDK-specific method only when the project contains a confirmed SDK artifact
   and version; an `operationId` is not an SDK method. Keep `X-API-Key` injection
   server-side through the Authentication handoff without exposing a real key.
6. Treat mutating timeouts or lost responses as ambiguous outcomes. Never
   blindly replay a create, delete, update, participant action, invite-link
   reset, join-request action, or message send. Any outbox, idempotency key,
   retry budget, or reconciliation job is project architecture unless the
   generated runtime reference says otherwise.

## Accepted versus final state

Create, delete, settings update, and participant removal return
`WhatsappGroupAsyncResponse`: HTTP `200`, `status=pending`, and `requestId` mean
only that YCloud accepted the asynchronous request. Preserve `requestId` for
correlation with the separately confirmed group webhook event; do not label the
group created/deleted/updated or participants removed at acceptance time.

Join-request approve/reject responses are itemized processing results; preserve
approved/rejected IDs, failed items, and unknown errors. Invite-link reset returns
the new link synchronously but says nothing about message delivery. Invite-link
message HTTP `200` returns `WhatsappMessage` and means message-request acceptance,
not delivery. Send its YCloud message ID and accepted state to
`ycloud-whatsapp-messages`, which owns retrieve/status handling and the
accepted-versus-final boundary. Webhook endpoint management and receiver
reliability belong to `ycloud-webhook-endpoints`.

## Mutation safeguards and tests

Deletes, participant removals, join-request decisions, invite-link resets, and
settings changes can be disruptive or irreversible. Resolve exact target IDs
and effects before acting. Proceed under the current request or existing
authorization; ask only for missing scope or newly uncovered effects. Use
no-network tests for local code and reconcile ambiguous live results.

Tests should cover every selected route and schema plus relevant negative cases:
path asymmetry, E.164 validation, subject/description limits, join mode default,
cursor bounds/opacity, empty or mixed join-request outcomes, partial failures,
participant count and exactly-one identifier rules, unknown properties/statuses,
provider error/request-ID mapping, and accepted-versus-final correlation.

For invite-link messages, executable tests must cover neither recipient field,
each field alone, both valid fields selecting `to`, valid `to` with malformed or
wrong-typed `recipient` behaving exactly like `to` alone, invalid `to` with valid
`recipient` rejecting without fallback, the required `group_id` parameter, and
the fact that acceptance is not delivery.

## Outcome requirements

Return the matched operation IDs and exact method/paths, authority-labeled
contract facts, project-local artifacts or proposed seams, no-network test
evidence, asynchronous correlation behavior, and explicit unknowns. Do not
claim an operation implemented because only a route label, button, sample JSON,
or mock response exists; link each implemented row to its adapter/handler and
behavioral tests.

### CANNOT

List unsupported operations, missing project facts, source conflicts, unconfirmed
SDK behavior, secret disclosure, out-of-scope access, automatic replay, final-state assumptions, and delivery claims.
Do not move documented async `pending`/`requestId` behavior or known partial
join-request results into `CANNOT`.

### Handoff

Send ordinary group messages and invite-link-message post-acceptance tracking to
`ycloud-whatsapp-messages` with the YCloud message ID kept separate from group,
request, and project correlation IDs. Send Flow lifecycle to
`ycloud-whatsapp-flows`, authentication storage to
`ycloud-api-authentication`, and webhook endpoint/receiver work to
`ycloud-webhook-endpoints`. Return to Architect with capability-row status,
changed or proposed artifacts, tests/results, accepted-versus-final evidence,
unknowns, and outgoing handoffs.
