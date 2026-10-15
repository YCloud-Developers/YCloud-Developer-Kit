---
name: ycloud-whatsapp-phone-numbers
description: Execute, design, implement, or evaluate YCloud WhatsApp business phone-number inventory, registration, profile, display-name, Business Username, contact-book, Calling/capture settings, and commerce settings operations. Use for phone-number management; exclude WABA management, message sending, template lifecycle, WhatsApp Calling sessions.
---

# YCloud WhatsApp Phone Numbers

Manage WhatsApp business phone numbers or design, implement, and evaluate
the fifteen operations in the generated reference.

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

Load [the generated OpenAPI reference](references/openapi.md) and
[the reviewed runtime reference](references/runtime.md) only after this skill is
selected. If retry, idempotency, queueing, or error translation is requested,
also read `references/shared/integration-boundaries.md`. Exact paths, schemas,
responses, and descriptions come from `openapi.md`; cross-cutting pagination,
error, request-ID, rate-limit, and compatibility behavior comes from
`runtime.md`. If either generated reference is missing, stale, or inconsistent
with its recorded source hash and operation count, report drift and stop rather
than reconstructing the contract from memory.
For phone-number list work, also read
[`references/shared/pagination-contract.md`](references/shared/pagination-contract.md).

## Exact operation scope

| Intent | Method and path | operationId |
| --- | --- | --- |
| List phone numbers | `GET /whatsapp/phoneNumbers` | `whatsapp_phone_number-list` |
| Retrieve phone number | `GET /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}` | `whatsapp_phone_number-retrieve` |
| Retrieve Business Username | `GET /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}/businessUsername` | `whatsapp_phone_number-retrieve-business-username` |
| Update Business Username | `PATCH /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}/businessUsername` | `whatsapp_phone_number-update-business-username` |
| Delete active Business Username | `DELETE /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}/businessUsername` | `whatsapp_phone_number-delete-business-username` |
| Retrieve username suggestions | `GET /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}/businessUsername/suggestions` | `whatsapp_phone_number-retrieve-business-username-suggestions` |
| Delete Meta contact-book entry | `DELETE /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}/contactBook/{bsuid}` | `whatsapp_phone_number-delete-contact-book-entry` |
| Update display name | `PATCH /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}/displayName` | `whatsapp_phone_number-update-displayName` |
| Retrieve profile | `GET /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}/profile` | `whatsapp_phone_number-retrieve-profile` |
| Update profile | `PATCH /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}/profile` | `whatsapp_phone_number-update-profile` |
| Register phone number | `POST /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}/register` | `whatsapp_phone_number-register` |
| Retrieve Calling/capture settings | `GET /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}/settings` | `whatsapp_phone_number-retrieve-settings` |
| Save Calling/capture settings | `POST /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}/settings` | `whatsapp_phone_number-save-settings` |
| Retrieve commerce settings | `GET /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}/whatsappCommerceSettings` | `whatsapp_phone_number-retrieve-commerce-settings` |
| Update commerce settings | `PATCH /whatsapp/phoneNumbers/{wabaId}/{phoneNumber}/whatsappCommerceSettings` | `whatsapp_phone_number-update-commerce-settings` |

WABA discovery belongs to `ycloud-whatsapp-business-accounts`. Message
submission/retrieval belongs to `ycloud-whatsapp-messages`; template lifecycle
and analytics belongs to `ycloud-whatsapp-templates`; authentication-only work
belongs to `ycloud-api-authentication`; broad multi-domain planning belongs to
`ycloud-integration-architect`. Readiness, repository maintenance, issue
tracking, WhatsApp Calling sessions, and contact CRUD are out of scope.

## Contract-first workflow

1. Match only allowlisted operations and report exact methods, paths,
   operationIds, parameters, request/response schemas, and documented responses.
   Treat `operationId` and `x-*` fields as identifiers or codegen hints, not SDK
   methods or business rules. Use an SDK-specific method only when its artifact,
   version, and documentation are confirmed in the project.
2. Preserve `wabaId`, YCloud phone-number IDs, and BSUIDs as opaque,
   case-sensitive strings; do not parse prefixes or coerce numeric-looking IDs.
   Preserve `phoneNumber` as an E.164 string, never a number. Encode path
   segments correctly; the contact-book contract explicitly requires the
   leading `+` to be encoded as `%2B` when constructing that path manually.
   Never infer that a phone belongs to a WABA: use only confirmed input or a
   WABA/phone lookup result.
3. For list, use 1-based `page`, `limit` from 1 through 100, and documented
   defaults. Request `includeTotal=true` only when a count is needed. Preserve
   `filter.wabaId` exactly; the contract says it is required when the account has
   more than 100 WABAs. Parse the response as the merged Page envelope with
   required `offset`, `limit`, `length`, phone-number `items`, and optional
   `total`; response `offset` is not a query parameter. Preserve unknown
   response fields and enum/status values.
4. Keep contract behavior distinct across operation families:

   - **Business Username:** PATCH sends a required plain username without `@`.
     Apply all generated length, character, letter, dot, prefix, and suffix
     constraints after the documented trim/lowercase normalization. A successful
     request may remain `reserved`; `pending_review` is a legacy response value,
     and an existing active username may coexist with the requested one. DELETE
     removes only the active username and does not cancel a reserved request.
     Suggestions flatten to `data: string[]`; no suggestions is an empty array.
   - **Meta contact book:** require a standard BSUID matching the generated
     shape; parent `.ENT.` BSUIDs are unsupported. The WABA, phone binding, and
     Meta business portfolio must match. This endpoint requires an account API
     key, not a Developer App key, but credential selection/storage remains an
     Authentication handoff. HTTP 200 always has `success=true`;
     `deleted=false` is a successful no-match, not a 404. The operation does not
     delete YCloud Contact/message/BSUID records, bypass Meta's 30-day cache, or
     prevent later recreation after another WhatsApp interaction.
   - **Profile and display name:** preserve requiredness and every generated
     field constraint exactly. Do not make `newName` required merely because the
     display-name endpoint's purpose suggests it if the schema does not. For
     profile updates, enforce field length, website count/item length, URL
     scheme, vertical values, and description-only `about` constraints without
     inventing replacement semantics for omitted fields.
   - **Registration:** preserve the no-body POST contract. Do not interpret a
     200 registration response as message readiness, template approval, or
     authorization to send.
   - **Calling/capture settings:** GET accepts optional `type=capture|calling`;
     omitted `type` follows the documented Calling response behavior. Save
     `calling`, `capture`, or both. When both are sent, each branch is attempted
     independently after shared authorization/phone validation; an error can
     mean the other branch was already saved. Enabling either capture switch
     requires the documented announcement language and purpose. Model combined
     failures as partial/ambiguous outcomes, not atomic rollback.
   - **Commerce settings:** preserve the two optional booleans and exact PATCH
     response shape. Do not add an undocumented catalog/cart dependency or infer
     omitted-field behavior.
5. Keep contract facts separate from local policy. Validation, confirmation UX,
   audit records, idempotency ledgers, retry budgets, and rollback controls are
   application-owned unless the references say otherwise. Both DELETEs require exact targets
   and effects covered by the current request or existing authorization. Treat mutating timeouts and the
   combined-settings failure as ambiguous; never replay a mutation
   automatically. Honor documented `Retry-After` before later traffic without
   treating it as proof that replay is safe.
6. Use placeholders such as `<YCLOUD_API_KEY>`, `<WABA_ID>`,
   `<E164_PHONE_NUMBER>`, and `<STANDARD_BSUID>` in examples and tests. For
   live work, use scoped identifiers and inspect only the needed resource data.
   Keep credential injection server-side through `ycloud-api-authentication`.

## Validation and evidence

For local implementation, add no-network tests for all selected operation
routes, exact path/query/body construction, opaque IDs, E.164 string handling
and path encoding, pagination boundaries/defaults/optional totals, the
more-than-100-WABA filter condition, standard errors/request IDs, and unknown
response properties or statuses. Add family-specific tests for username
normalization and validation, active-versus-reserved state, empty suggestions,
contact-book account-key gating and `deleted` semantics, profile/display-name
constraints, no-body registration, settings `type`, capture prerequisites,
combined-settings partial failure, commerce booleans, delete target/authorization checks,
ambiguous timeouts, and no mutation replay. Mocks must prove no network client is
invoked.

Return these sections, adapted to the requested deliverable:

1. **Matched contract** — source hash, selected operations, exact request and
   response shapes, and description-only constraints.
2. **Construction or implementation** — placeholder HTTP/project-local design,
   changed artifacts, and project facts still needed.
3. **Contract versus policy** — identify YCloud guarantees separately from local
   validation, approvals, persistence, retries, and rollback choices.
4. **Tests and evidence** — synthetic cases and actual no-network results.
5. **CANNOT** — out-of-scope actions, secret disclosure, guessed SDK
   methods, unsupported operations, unconfirmed retry/idempotency/atomicity, and
   missing facts. Do not put confirmed pagination, error-envelope, request-ID,
   status, or partial-success behavior in `CANNOT`.
6. **Handoff** — receive a confirmed opaque WABA ID from
   `ycloud-whatsapp-business-accounts`; once the WABA/phone pair is confirmed,
   hand message submission/retrieval to `ycloud-whatsapp-messages` and template
   lifecycle/analytics to `ycloud-whatsapp-templates`. Do not imply that phone
   registration or configuration authorizes either downstream mutation. Return
   to Architect with capability status, artifacts, tests, unknowns, and outgoing
   handoffs.

## Execution safeguards

Execute only within the account, environment, targets, and effects covered by
the current request or prior authorization. Ask for an unresolved material
boundary, not for repeated approval of the same operation. Verify live results
and keep mock evidence separate; never blindly replay an ambiguous mutation.
