---
name: ycloud-custom-events
description: Execute, design, implement, or evaluate YCloud custom-event definition/property lifecycle and CONTACT-associated event ingestion. Use for the seven allowlisted Custom Events operations; exclude Contact lifecycle, webhook consumption, analytics.
---

# YCloud Custom Events

Manage custom-event definitions and properties and ingest CONTACT-associated
events, or design, implement, and evaluate their integration.

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

After this Skill is selected, read the pinned [OpenAPI
contract](references/openapi.md) and reviewed [runtime
boundaries](references/runtime.md). If retry, idempotency, outbox/queue, rate
limiting, or error translation is requested, also read
`references/shared/integration-boundaries.md`. If the source hash, operation
coverage, or contract facts drift, stop and report the mismatch instead of
guessing.

## Exact scope and routing

Definition and property lifecycle is separate from occurrence ingestion:

| Concern | Method and path | operationId |
| --- | --- | --- |
| Create definition | `POST /event/definitions` | `custom_events-create-definition` |
| Retrieve definition | `GET /event/definitions/{name}` | `custom_events-retrieve-definition` |
| Update definition metadata | `PATCH /event/definitions/{name}` | `custom_events-update-definition` |
| Create property definition | `POST /event/definitions/{name}/properties` | `custom_events-create-property-definition` |
| Delete property definition | `DELETE /event/definitions/{name}/properties/{propertyName}` | `custom_events_delete-property-definition` |
| Update property metadata | `PATCH /event/definitions/{name}/properties/{propertyName}` | `custom_events_update-property-definition` |
| Ingest event occurrence | `POST /event/events` | `custom_events-send-event` |

The first six operations manage reusable definitions; they do not ingest an
event. The final operation submits one occurrence; it does not create or alter
its definition. Contact create/retrieve/update/delete and identity resolution
belong to the Contact workflow. Webhook consumption, analytics, readiness, and
broad multi-domain planning are outside this Skill.

## Contract-first workflow

1. Match only the selected operations above to their exact method, path,
   parameters, request schema, responses, and descriptions in `openapi.md`.
   Treat `operationId` as an identifier, not an SDK method. Use raw HTTP shapes
   or an SDK artifact/version already confirmed in the project.
2. For definition creation, preserve required `name`, `label`, and `objectType`;
   the pinned enum contains only `CONTACT`. Preserve the exact source patterns
   and lengths without “correcting” or anchoring them. Definition update changes
   only `label` and/or `description` in the declared request shape.
3. For property creation, preserve required `name`, `label`, and `type`, and the
   declared types `STRING`, `NUMBER`, `TIMESTAMP`, and `URL`. Property update
   changes only `label` and/or `description`; changing a property's name or type
   is not an allowlisted update behavior. For deletion, identify the exact definition/property pair and effect; proceed
   when covered by the request or existing authorization.
4. Before ingestion, require an already-defined `eventName` and validate event
   property names and values against the confirmed definition. The source says
   NUMBER accepts numeric values with up to one decimal, TIMESTAMP accepts epoch
   milliseconds, and URL accepts strings beginning with `http://` or `https://`.
   Do not infer coercion, undeclared-property handling, schema evolution, or
   server-side validation behavior beyond the pinned descriptions.
5. Use contact identity supplied by the user, scoped prior evidence, or a
   Contact handoff. Accept a confirmed opaque contact ID for `objectId` or a
   confirmed phone number for `contactPhoneNumber`, preserving its provenance.
   When lookup is needed, route it internally to Contacts under the existing
   authorization; do not require the user to produce a handoff artifact. Choose one association form as project policy; the OpenAPI says
   the phone number is an alternative but does not define precedence when both
   fields are present. If both association forms are supplied without a clear choice, resolve it
   from context or ask for that material decision; do not guess precedence.
6. Keep `occurTime` as RFC 3339 when supplied. The source says the current time
   is used when omitted, but does not define whose clock, timezone normalization,
   or replay semantics. Preserve omission intentionally and test supplied and
   omitted branches with a fake clock where project code needs deterministic
   behavior.
7. Treat a documented HTTP `200` as synchronous acceptance/success for the
   selected endpoint only. For event ingestion, the source declares no response
   body, event ID, processing state, callback, query operation, ordering rule,
   or downstream-completion guarantee. Never label an accepted occurrence as
   processed, delivered, or completed.
8. Execute the selected operations under the shared execution policy. Treat a timeout
   or lost response from create, update, delete, or send as an ambiguous outcome.
   Never replay automatically. Any idempotency key, outbox identity, deduplication
   ledger, retry budget, or reconciliation process is project-owned architecture
   unless a separate authoritative contract confirms it.

## Illustrative ingestion shape

This is documentation only; do not execute it:

```http
POST <YCLOUD_API_BASE_URL>/event/events
X-API-Key: <YCLOUD_API_KEY>
Content-Type: application/json

{"eventName":"<DEFINED_EVENT_NAME>","objectId":"<CONFIRMED_CONTACT_ID>","occurTime":"<RFC3339_TIME>","properties":{"<DEFINED_PROPERTY>":"<SYNTHETIC_VALUE>"}}
```

Use `contactPhoneNumber` instead of `objectId` when that is the selected
confirmed identity from the request, prior context, or Contact handoff. Do not put a real key, contact, phone number, event, or
customer property in examples, fixtures, logs, or generated artifacts.

## Outcome requirements

Adapt the result to planning, implementation, execution, or evaluation and make these
items explicit:

1. **Matched contract** — source hash, selected operation IDs, exact methods and
   paths, parameters, request/response schemas, and description-only rules.
2. **Lifecycle versus ingestion** — definition/property changes and occurrence
   submission remain distinct in architecture, handlers, permissions, and tests.
3. **Contact handoff** — record whether the input is a confirmed opaque contact
   ID or confirmed contact phone number, its provenance, and any project-owned
   choice when both could exist; route any needed Contact lookup internally and
   perform no unrequested Contact mutation.
4. **Acceptance boundary** — distinguish endpoint `200` from unconfirmed
   downstream processing or completion and preserve unknown future fields.
5. **Safety** — trusted-server placement, protected credential injection,
   scoped authorization for deletion, timeout ambiguity, and no automatic replay.
6. **Tests** — exact routing and schemas; name/type constraints; CONTACT-only
   definition handling; update field allowlists; property-value validation;
   Contact artifact branches; supplied/omitted occurrence time; empty-body `200`;
   `404` lifecycle fixtures; ambiguous timeout/no replay; and proof of no network.
7. **CANNOT** — secret disclosure, out-of-scope Contact mutation,
   undocumented property coercion/precedence, safe replay or exactly-once claims,
   downstream status/completion, analytics, callbacks, guessed SDK methods, and
   unlisted errors or operations.
8. **Handoff** — send contact creation, retrieval, normalization, or identity
   choice to Contacts when needed; preserve supplied or retrieved identity evidence. Return to Architect
   with capability-row status, artifacts, tests/results, accepted-versus-final
   evidence, unknowns, and outgoing handoffs.

## Safety and source priority

Use the pinned OpenAPI source first, these derived references second, and this
workflow third. Keep source facts, observed runtime evidence, and project policy
visibly separate. Use the shared execution policy for live operations and
synthetic fixtures for local verification.
