---
name: ycloud-integration-architect
description: Execute, design, implement, or evaluate a cross-domain YCloud integration from task and project context, with explicit scope, deliverable, authorization boundaries, domain handoffs, and coverage evidence. Use for broad integration or multi-domain orchestration; do not use for readiness, repository-maintenance, issue-tracker, support, or requests confined to one domain.
---

# YCloud Integration Architect

Orchestrate cross-domain YCloud design, implementation, evaluation, and real
operations. Route domain contract decisions to the matching Skill and merge
their results, preserving the user’s request and existing authorization.

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

## Normalize the request

Resolve these dimensions from the user's words and project context:

```text
scope: focused | full-first-wave | full-whatsapp-operations | full-supported-operations
deliverable: project-integration | test-console | production-plan | live-operation
mutation: read-only | local-write-authorized | live-read-authorized | live-write-authorized
authorization_scope: account, environment, targets, operations, credential reference
```

- Default to `focused` when the user names a concrete outcome or domain.
- “All currently supported capabilities” means `full-supported-operations`: 88
  API operations plus 2 cross-cutting capabilities = 90 coverage rows.
- `full-first-wave` remains a compatibility mode: 17 API operations + 2
  cross-cutting capabilities = 19 coverage rows.
- “All YCloud APIs” requires an explicit `88 API operations + 2 cross-cutting capabilities = 90 coverage rows`
  disclosure, with 7 operations excluded by product decision. Product exclusion
  is not provider deprecation; never claim all 95 are covered.
- Select `test-console` only when the user asks for a test UI, console, explorer,
  or comparable interactive surface. Do not turn `full-first-wave` into a
  dashboard on its own.
- Select `production-plan` for production architecture or rollout planning. It
  is read-only unless the user separately asks to implement local artifacts.
- Select `live-operation` for a requested real query or action. Derive live-read
  or live-write authorization from the request and prior grants; do not ask the
  user to restate it in these labels. Carry the resolved scope across domains.
- Local implementation alone permits project work, not incidental live effects.
  A request combining implementation and a live check can authorize both.
  Keep account/environment and target limits explicit without adding an approval
  step when those facts are already available.

Read only the selected mode reference:

- `focused` → [references/modes/focused.md](references/modes/focused.md)
- `full-first-wave` → [references/modes/full-first-wave.md](references/modes/full-first-wave.md)
- `full-whatsapp-operations` → [references/modes/full-whatsapp-operations.md](references/modes/full-whatsapp-operations.md)
- `full-supported-operations` → [references/modes/full-supported-operations.md](references/modes/full-supported-operations.md)
- `test-console` → [references/modes/test-console.md](references/modes/test-console.md)
- `production-plan` → [references/modes/production-plan.md](references/modes/production-plan.md)
- `live-operation` → [shared execution policy](references/shared/integration-boundaries.md#execution-and-authorization); use focused routing for the selected operations

When scope or completion is material, also read
[references/capability-catalog-summary.md](references/capability-catalog-summary.md)
and [references/coverage-matrix-schema.md](references/coverage-matrix-schema.md).
For multi-domain work, read
[references/handoff-contracts.md](references/handoff-contracts.md).

## Domain routing

| Concern | Owner Skill | Boundary |
| --- | --- | --- |
| API key, trusted-server header, storage, API-key rotation plan | `ycloud-api-authentication` | Use configured credentials without exposing them; resolve ambiguous rotation targets |
| Direct/queued message send or retrieve | `ycloud-whatsapp-messages` | Existing-template sending belongs here; accepted is not final |
| WhatsApp media upload | `ycloud-whatsapp-media` | Return a media handoff; upload is not send |
| Template lifecycle and analytics | `ycloud-whatsapp-templates` | Lifecycle is separate from template sending |
| Webhook endpoint management or receiver | `ycloud-webhook-endpoints` | Endpoint management and event receipt remain separate |
| WABA inventory or ACO settings | `ycloud-whatsapp-business-accounts` | Pass opaque WABA identity to Phone Numbers/Messages/Templates |
| Phone registration, profile, username, settings, or commerce | `ycloud-whatsapp-phone-numbers` | Calling settings do not include WhatsApp Calling sessions |
| Group lifecycle, membership, invite links, or settings | `ycloud-whatsapp-groups` | Group management and message final state remain separate |
| Flow lifecycle, preview, publish, deprecate, or delete | `ycloud-whatsapp-flows` | Flow management and Flow message sending remain separate |
| Mark inbound message read or show typing | `ycloud-whatsapp-inbound-messages` | Consume a verified Receiver message identity, not event ID |
| Account balance | `ycloud-balance` | Balance is readiness evidence, not a send/delivery guarantee |
| Contact CRUD, attributes, or notes | `ycloud-contacts` | Contact and note IDs remain distinct; writes are not replay-safe by default |
| Custom event definitions, properties, or ingestion | `ycloud-custom-events` | Definition lifecycle and event acceptance remain separate |
| Customer/channel unsubscribe state | `ycloud-unsubscribers` | Eligibility evidence is not provider send or delivery state |
| WhatsApp Calling session commands or call media | `ycloud-whatsapp-calling` | Command acceptance is not final call state; media is a separate handoff |

Read `references/openapi.md` and `references/runtime.md` for Architect-level
provenance, then load only the selected domain's narrow OpenAPI/runtime
references. Never load the full OpenAPI snapshot by default. For error
translation, retry, idempotency, rate limiting, webhook reliability, or
cross-domain policy, read `references/shared/integration-boundaries.md`. Stop on
source conflict or drift instead of guessing.
When any selected operation lists resources, also read
`references/shared/pagination-contract.md` and preserve its operation-specific
response envelope through client, service, handler, and UI-facing DTO tests.

When the user explicitly asks for sandbox/mock/no-real-side-effect integration
testing or a local YCloud base-URL replacement, also read
`references/shared/sandbox-contract.md`. Keep its provider-shaped surface,
Developer Kit policy, and mock-only control namespace separate; do not count the
Facade as additional OpenAPI coverage or silently substitute it for a real
production-readiness check.

When the user asks for a Java/Node reference project, SDK/Quickstart support,
generated-project evaluation, TTPRI, Beta evidence, or release readiness, also
read `references/shared/reference-integrations.md`. Treat executable paths as
source-distribution assets that may be absent from an installed Plugin. Keep
oracle harness results, caller-supplied candidate evidence, source-bound
receipts, synthetic TTPRI and real Beta evidence as separate trust layers.

## Project work

Inspect only non-secret project structure needed to identify runtime, framework,
build files, server/client boundary, configuration pattern, HTTP client, tests,
and an existing YCloud seam. Reuse the project's architecture and UI framework;
do not introduce React, Vue, Next.js, Spring, or another framework merely because
a test console was requested.

When local writes are authorized, implement the smallest complete vertical seam
for the selected scope and run proportionate no-network tests. Preserve unrelated
and concurrent edits. When writes are not authorized, provide a plan with
concrete project seams and do not change files. A request to implement or fix
the scoped project is sufficient local-write authorization; do not ask again.
Direct live operations do not require a local project or generated code.

## Contract rules

- Exact paths, methods, schemas, and descriptions come from the selected
  generated reference. Use placeholders and synthetic IDs in examples/tests,
  and scoped real identifiers for authorized operations.
- `operationId`, codegen extensions, generated model names, and `allOf` do not
  establish an SDK method or new provider behavior.
- Use `runtime.md` for the documented error envelope, request ID, rate limits,
  pagination, compatibility, and asynchronous behavior. Keep provider contract,
  Developer Kit policy, and project decisions visibly separate.
- Include a provider error/request-ID/rate-limit adapter whenever selected
  operations need those cross-cutting runtime behaviors.
- Generate tolerant clients: preserve unknown properties and enum/event values,
  keep explicit unknown response/enum compatibility handling, and treat YCloud
  IDs as opaque case-sensitive strings up to 255 characters.
- Do not invent general idempotency, replay safety, fixed signature tolerance,
  delivery guarantees, or unlisted errors. Treat ambiguous mutation timeouts as
  ambiguous outcomes.

## Orchestration and evidence

Send each owner a handoff containing mode, selected capability IDs, project
seams, authorization level, preconditions, and evidence expected. Require the
domain result to return implemented/deferred/blocked rows, changed artifacts,
tests and results, unknowns, and any outgoing handoff. Merge those results
yourself; do not finish with instructions for the user to invoke every domain
Skill manually.

For `full-first-wave`, account for all 19 rows. For
`full-whatsapp-operations`, account for all 62 rows. For
`full-supported-operations`, account for all 90 rows. A row is not implemented merely
because a menu, card, route name, sample JSON, or button exists. Implemented rows
need a linked adapter/handler/service artifact and behavioral test evidence.
Deferred and blocked rows are allowed only with explicit reasons; silent omissions
fail coverage. Future and product-excluded operations remain disclosure rows and
must never be counted as implemented coverage.

## Outcome requirements

Adapt the response to the selected deliverable instead of forcing fixed headings.
Always make these facts easy to verify:

- normalized scope, deliverable, and mutation level;
- confirmed project facts and source provenance;
- selected capability rows and domain owners;
- architecture and cross-domain handoffs;
- local changes, proposed seams, or observed live results, as applicable;
- tests/evidence and accepted-versus-final state boundaries;
- deferred, blocked, unknown, unsupported, and authorization-gated items;
- next action needed from the user, if any.

### CANNOT

Keep secret values, unrelated customer data, unconfirmed SDK behavior,
unsupported operations, and invented provider guarantees out of the result.
Check exact targets and effects for destructive or disruptive actions, execute
within existing authorization, and verify outcomes. An ambiguous mutation
requires reconciliation, not blind replay or a fabricated success claim.

### Handoff

Return a merged coverage/evidence result to the user. If further work requires a
new scope or a missing material decision not covered by existing authorization,
identify the owner and ask for only that authorization or fact. Continue
already-authorized work without restarting approval at domain handoffs.
Do not substitute a list of Skills the user must invoke for the merged result.

Use placeholders and synthetic data in examples/tests. Installation readiness
belongs to the smoke Skill; requested live checks use the domain execution policy.
Repository maintenance, issue tracking, ordinary product explanation, and
support cases remain outside this Skill.
