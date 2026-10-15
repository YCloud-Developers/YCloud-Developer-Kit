---
name: ycloud-unsubscribers
description: Execute, design, implement, or evaluate YCloud customer unsubscribe creation, lookup, listing, and deletion. Use for opt-out records and message-eligibility evidence; exclude Contact CRUD, message sending, provider delivery status.
---

# YCloud Unsubscribers

For `unsubscriber-list`, read
[`references/shared/pagination-contract.md`](references/shared/pagination-contract.md)
with the generated OpenAPI/runtime references.

Manage customer unsubscribe records or design, implement, and evaluate their
integration using the five operations in the pinned reference.

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

After this Skill is selected, read the generated [OpenAPI
contract](references/openapi.md) and reviewed [runtime
behavior](references/runtime.md). For retry, idempotency, error translation, or
production architecture, also read
`references/shared/integration-boundaries.md`. If either local reference is
missing, stale, or inconsistent with its recorded source hash or operation
count, report drift and stop rather than reconstructing the contract from
memory.

## Exact operation scope

| Intent | Method and path | operationId |
| --- | --- | --- |
| Create an unsubscribe record | `POST /unsubscribers` | `unsubscriber-create` |
| Delete by composite identity | `DELETE /unsubscribers/{customer}/{channel}` | `unsubscriber-delete-by-customer-and-channel` |
| List unsubscribe records | `GET /unsubscribers` | `unsubscriber-list` |
| List all records for one customer | `GET /unsubscribers/{customer}` | `unsubscriber-list-all-by-customer` |
| Retrieve by composite identity | `GET /unsubscribers/{customer}/{channel}` | `unsubscriber-retrieve-by-customer-and-channel` |

Contact discovery and updates belong to a Contact capability; message request
construction, `filterUnsubscribed`, submission, retrieval, and message status
belong to `ycloud-whatsapp-messages`. Authentication belongs to
`ycloud-api-authentication`; broad multi-domain planning belongs to
`ycloud-integration-architect`. Webhook processing, keyword configuration,
marketing consent design, and non-WhatsApp channel behavior are out of scope.

## Contract-first workflow

1. Match only the five allowlisted operations. Report exact method, path,
   `operationId`, parameters, body, response schema, and documented status.
   Treat operation IDs and `x-*` fields as identifiers or codegen hints, not SDK
   method names or additional behavior.
2. Model `(customer, channel)` as the unique Unsubscriber identity. Do not use
   `regionCode`, `source`, or `createTime` as identity. For the documented
   `type=PHONE_NUMBER`, require `customer` to remain an E.164 string, never a
   number, and URL-encode it as one path segment (including the leading `+`).
   The only currently documented channel is lowercase `whatsapp`; do not
   normalize an unknown future channel into it. Consume a Contact handoff only
   when it supplies a confirmed customer value and its identity provenance;
   never infer an E.164 number from a contact name or unrelated identifier.
3. For create, send required `type`, `customer`, and `channel`; `regionCode` is
   optional. Preserve the documented `PHONE_NUMBER` and `whatsapp` values, but
   decode future `type`, `channel`, and `source` values through an explicit
   unknown branch. Preserve unknown response properties. Do not interpret the
   response `source` prose as an additional create/delete guarantee.
4. For `unsubscriber-list`, preserve `page` as 1-based with range 1-100,
   `limit` as 1-100, defaults of 1 and 10, optional `includeTotal`, optional
   `pageAfter`, and exact dotted filters `filter.customer`, `filter.channel`,
   and `filter.regionCode`. Parse the successful response as the merged Page
   envelope with required `offset`, `limit`, `length`, resource `items`, optional
   `total`, and optional `cursor`; `offset` is response metadata. Treat `total`
   as optional and present only when
   requested. Cursor traversal must use the returned opaque `cursor.after`
   unchanged and stop when it is absent. Add project-owned safety guards for an
   empty page, a repeated cursor, and a configured page/item budget. Do not
   synthesize a cursor from offsets, infer completion from `total`, or silently
   switch/mix page and cursor strategies beyond behavior confirmed by the
   contract.
5. Keep `GET /unsubscribers/{customer}` distinct: it returns an array, not an
   `UnsubscriberPage`, and documents `404`. Composite retrieve and delete also
   document `404`; list and create declare only `200`. Use reviewed runtime
   behavior for generic errors and do not invent endpoint-specific statuses.
6. Keep provider contract separate from local policy. Confirmation UX, local
   consent rules, audit retention, eligibility caches, retry budgets, and
   reconciliation are application-owned. A timeout or lost response to create
   or delete has an ambiguous outcome. The contract defines no general
   idempotency key, so never replay either mutation automatically. Honor
   `Retry-After` before later traffic without treating it as replay authority.
7. Use placeholders such as `<YCLOUD_API_KEY>` and `<E164_CUSTOMER>` in
   synthetic fixtures. For live work, use scoped customer/channel identities
   and inject credentials server-side through Authentication; do not disclose
   keys or retrieve unrelated business data.

## Eligibility-policy evidence

An exact composite retrieve, a customer-wide list, or a deliberately complete
and defensively traversed filtered list can produce an evidence object for the
Messages capability. Include the queried customer/channel, match or no-match,
operation and source provenance, observation time/freshness, pagination
completeness, and unknown-value warnings. A local cache entry or incomplete
page traversal is not authoritative negative evidence.

This evidence is an input to the application's message-eligibility policy. It
does not submit or suppress a message by itself and is not a YCloud/WhatsApp
final status. Never label it sent, accepted, queued, failed, delivered, read, or
provider-rejected. Messages owns request-time policy (including whether to use
`filterUnsubscribed`) and provider response/webhook interpretation.

## Validation and evidence

For local implementation, add no-network tests for exact routes, required body
fields, E.164 string preservation and path encoding, composite identity,
array-versus-complete-page-envelope response shapes, all list parameters and dotted filters,
optional total, opaque cursor continuation, absent/repeated cursor termination,
empty pages, configured traversal budgets, `404` handling, standard errors and
request IDs, and unknown fields/enum values. Add mutation tests for explicit
authorization, ambiguous outcomes, and proof of no automatic replay. Mocks must
prove that no network client is invoked.

Return these sections, adapted to the requested deliverable:

1. **Matched contract** — source hash, selected operations, exact request and
   response shapes, and description-only constraints.
2. **Construction or implementation** — placeholder design, changed artifacts,
   and project facts still needed.
3. **Contract versus policy** — provider facts separated from local consent,
   eligibility, cache, pagination-budget, retry, and audit choices.
4. **Tests and evidence** — synthetic cases, actual no-network results, and
   pagination completeness.
5. **CANNOT** — out-of-scope actions, secret disclosure, unsupported operations,
   guessed SDK methods, automatic mutation replay, unconfirmed idempotency, or
   provider send/delivery/final-status claims.
6. **Handoff** — consume confirmed customer identity evidence from Contact;
   produce provenance-bearing eligibility-policy evidence for
   `ycloud-whatsapp-messages`; return capability status, artifacts, tests,
   unknowns, and outgoing handoffs to Architect. Do not claim a Contact exists
   merely because an Unsubscriber exists, or that an eligible result guarantees
   a message will be accepted or delivered.

## Execution safeguards

Execute only within the account, environment, targets, and effects covered by
the current request or prior authorization. Ask for an unresolved material
boundary, not for repeated approval of the same operation. Verify live results
and keep mock evidence separate; never blindly replay an ambiguous mutation.
