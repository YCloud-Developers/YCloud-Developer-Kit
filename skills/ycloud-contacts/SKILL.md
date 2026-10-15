---
name: ycloud-contacts
description: Execute, design, implement, or evaluate YCloud Contacts lifecycle, attribute, and note operations with strict contact/note ID separation and defensive pagination. Use for Contacts API work; exclude Custom Events, Unsubscribers, message sending.
---

# YCloud Contacts

Manage contacts, contact attributes, and contact notes, or design, implement,
and evaluate their integration. A contact change does not itself perform a
downstream event, subscription, or message action.

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
all ten operations below, stop and report the drift.

## Exact operation allowlist

Load both generated references after this Skill is selected. Match only these
operations and preserve every source-defined parameter, request/response schema,
status code, and description constraint.
For contact list work, also read
[`references/shared/pagination-contract.md`](references/shared/pagination-contract.md).

| Intent | Method and path | operationId |
| --- | --- | --- |
| List attributes | `GET /contact/contacts/attributes` | `contact-attributes-list` |
| Create contact | `POST /contact/contacts` | `contact-create` |
| Delete contact | `DELETE /contact/contacts/{id}` | `contact-delete` |
| List contacts | `GET /contact/contacts` | `contact-list` |
| Create note | `POST /contact/contacts/{contactIdentifier}/notes` | `contact-note-create` |
| Delete note | `DELETE /contact/notes/{noteId}` | `contact-note-delete` |
| Update note | `PATCH /contact/notes/{noteId}` | `contact-note-update` |
| List notes | `GET /contact/contacts/{contactIdentifier}/notes` | `contact-notes-list` |
| Retrieve contact | `GET /contact/contacts/{id}` | `contact-retrieve` |
| Update contact | `PATCH /contact/contacts/{id}` | `contact-update` |

No search operation is in this allowlist. Contact attributes are configurations,
not contact values. Contact retrieve/list responses do not contain notes; use
`contact-notes-list` when note contents or note IDs are needed.

## Identifier separation

Treat every identifier as opaque and case-sensitive except where the contract
defines syntax. Never substitute one identifier class for another.

- Contact `{id}` and note-owner `{contactIdentifier}` accept a contact ID, an
  E.164 phone number beginning with `+`, or a Meta username without `@`, up to
  255 characters. Ambiguous numeric input resolves as a username first and as a
  contact ID only when no matching username exists; do not pre-resolve it with
  local numeric heuristics.
- Note `{noteId}` is a separate 24-character hexadecimal ObjectId. It is valid
  only for `contact-note-update` and `contact-note-delete`. Obtain note IDs from
  note create/list responses, never from the owning contact ID.
- A `ContactNote.contactId` identifies the owner and is not the note's `id`.
  For note mutations embedded in `contact-update`, an item without `id` creates
  a note; an item with a note ID updates an owned note. Omitted notes remain
  unchanged, an empty array changes nothing, and deletion uses the dedicated
  note-delete operation.

## Contract-first workflow

1. Select one allowlisted operation and a confirmed server-side project seam.
   Preserve unknown response properties and unknown attribute/source values;
   do not fail exhaustive decoding when YCloud adds compatible fields or enums.
2. Preserve request constraints. Contact create requires `phoneNumber`; notes
   trim surrounding whitespace and require 1–500 characters after trimming;
   contact create/update accepts at most 50 notes; tags accept at most 50 values
   of at most 50 characters. `nickname` is a deprecated input alias for
   `remarkName`, while response `nickname` is read-only WhatsApp data.
3. Keep contact updates patch-like. A non-null `customAttributes` array replaces
   all previous custom attributes. A no-op persisted-field update emits no
   `contact.attributes_changed` event, although note mutations in the request
   still apply. Do not infer note success from the returned `Contact`, because
   that schema excludes notes.
4. Use only synthetic contacts, phone numbers, usernames, note IDs, and payloads
   in examples and tests. Keep `X-API-Key` injection server-side through the
   Authentication handoff without exposing a real key. An `operationId` is not an
   SDK method name.
5. Treat a timeout or lost response for create, update, delete, or note mutation
   as ambiguous. Never replay it automatically. Note create has no client
   idempotency key and replay creates another note; repeated note delete returns
   `404`; repeated note update may emit another update event even when content is
   unchanged. Any reconciliation or deduplication design is project policy.

## Defensive pagination

Contact list supports two modes; choose one and do not silently switch modes.

- Page mode omits `pageAfter`; `page` is 1–100 with default 1, `limit` is 1–100
  with default 10, and `includeTotal` defaults false.
- Its successful wire response is the merged `ContactPage allOf Page` object:
  required `offset`, `limit`, and `length`; optional `total`; resource `items`;
  and optional `cursor`. `offset` is response metadata, not a request parameter.
  Preserve each Contact field, including `phoneNumber`, `countryCode`,
  `countryName`, `sourceType`, `lastSeen`, and `lastMessageToPhoneNumber`, plus
  unknown properties. Do not unwrap a nonexistent `data` property.
- Forward-cursor mode starts with the string `pageAfter=0`, then passes the exact
  returned `cursor.after` value unchanged. Never parse, increment, synthesize,
  or persist assumptions about cursor contents.
- Do not combine `pageAfter` with `page`, `pageBefore`, `offset`, or `sort`. Keep
  filters unchanged for the traversal. A missing `cursor` means there is no next
  page; do not assume an empty page, `length < limit`, or `total` is authoritative
  evidence of continuation.
- Cursor traversal is ordered by contact ID ascending and weakly consistent
  under concurrent inserts, deletes, and filter-field updates. Restart with
  `pageAfter=0` when a fresh traversal is required. `total`, when requested, is
  the complete filtered count and is not reduced by cursor position.

The notes-list endpoint is not cursor-paginated: it returns all notes, newest
first, with a maximum of 50. Do not add page parameters to it.

## Mutation safeguards and tests

Use no-network tests for local implementation. For a real mutation, resolve the
exact contact identifier or note ID and apply the existing authorization. Do
not ask again when the target and effect are already covered. Reconcile an
ambiguous result before considering another attempt.

Tests should cover all ten routes and methods, required and optional body shape,
contact/note ID non-interchangeability, username-first ambiguity, note limits and
trimmed content, incremental note semantics, custom-attribute replacement, the
complete provider-shaped ContactPage envelope and item mapping, pagination mode
conflicts and terminal cursor absence, unknown fields/enums,
provider error/request-ID mapping, and no automatic mutation replay.

## Outcome requirements

Return matched operation IDs and exact method/paths, authority-labeled contract
facts, project-local artifacts or proposed seams, no-network test evidence,
identifier handling, pagination mode, explicit unknowns, and handoffs. Do not
claim implementation from only a route label, sample JSON, or mock response;
link each implemented row to its adapter/handler and behavioral tests.

### CANNOT

List unsupported operations, missing project facts, source conflicts,
unconfirmed SDK behavior, secret disclosure, out-of-scope access, automatic mutation replay, inferred note ownership, or
downstream event/subscription/message claims. Do not move documented contact
identifier resolution, note-ID syntax, or cursor behavior into `CANNOT`.

### Handoff

Contacts is a producer for downstream capabilities, not their executor. Hand a
confirmed contact ID and the project-approved event payload to Custom Events;
hand a confirmed contact identity plus channel intent to Unsubscribers; hand a
confirmed E.164 recipient or supported recipient identity to Messages. Keep
contact IDs, note IDs, event IDs, unsubscriber records, and message IDs separate,
and do not infer that a contact mutation performed any downstream action.

Send authentication storage to `ycloud-api-authentication`. Return to Architect
with capability-row status, changed or proposed artifacts, tests/results,
pagination and identifier evidence, unknowns, and outgoing handoffs.
