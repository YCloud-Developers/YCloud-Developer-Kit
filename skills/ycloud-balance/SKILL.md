---
name: ycloud-balance
description: Execute, design, implement, or evaluate retrieval and interpretation of the YCloud account balance. Use for the single Balance read operation or a balance-to-readiness evidence handoff; exclude plugin readiness checks, billing history, top-ups.
---

# YCloud Balance

Retrieve the current account balance or design, implement, and evaluate its
integration. Report the provenance and freshness of observed balance data.

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

## Scope and handoffs

After this Skill is selected, read the generated [OpenAPI
contract](references/openapi.md) and reviewed [runtime
behavior](references/runtime.md). If retry, error translation, rate limiting, or
production architecture is requested, also read
`references/shared/integration-boundaries.md`. If either generated reference is
missing, stale, or conflicts with the pinned source, report the drift and stop
instead of guessing.

This Skill owns exactly `GET /balance`, operationId `balance-retrieve`. It does
not own transactions, billing history, invoices, spending forecasts, top-ups,
currency conversion, or account mutation. API-key configuration belongs to
`ycloud-api-authentication`; broad planning belongs to
`ycloud-integration-architect`.

Balance may provide one synthetic or observed-at-runtime input to an
application's operational-readiness policy, but this Skill does not define a
minimum sufficient balance and cannot certify plugin, deployment, account, or
production readiness. Route an explicit Developer Kit installation/readiness
check to the readiness/smoke workflow. Keep any application threshold, alert,
reservation, or fail-open/fail-closed rule labeled as project policy.

## Contract-first workflow

1. Confirm the source hash, exact method/path, `operationId`, lack of parameters
   and request body, and response schema in the generated reference. Do not
   infer an SDK method from `balance-retrieve`.
2. Preserve the documented `200 Balance` shape: required numeric `amount` and
   required string `currency`, where currency is an ISO 4217 code. Do not assume
   `amount` is an integer, minor units, non-negative, available credit, or a
   promise that a future operation will succeed. Preserve decimal precision
   according to the target project's established money strategy; if none
   exists, surface that decision instead of silently rounding through binary
   floating-point arithmetic.
3. Keep provider-generated identifiers and future opaque strings
   case-sensitive and unparsed. Preserve unknown response properties and an
   explicit unknown branch for future enum-like values. Do not convert currency
   or combine balances unless a separate, authoritative project contract is in
   scope.
4. At the provider adapter, retain the standard error envelope and
   `YCloud-Request-ID` (or `error.requestId`) for redacted correlation. The
   operation itself declares only `200`; use reviewed cross-cutting runtime
   behavior for generic failures and do not invent balance-specific status
   codes or error meanings. Branch on HTTP status and `error.code`, never
   diagnostic `error.message`.
5. On `429`, honor `Retry-After` before another request and parse beta
   `RateLimit-*` headers defensively. Do not assign an invented balance quota.
   Because retrieval is read-only, a bounded transient retry may be proposed as
   project policy, but no retry count, backoff, cache lifetime, or staleness
   tolerance is a YCloud guarantee unless `runtime.md` says so.
6. Keep the API key on a trusted server. Use placeholders and a fake adapter
   for local tests; label synthetic results. For a requested balance query,
   call the configured account and report amount, currency, and observation time.

## Illustrative raw HTTP

This shape is documentation only; do not execute it:

```http
GET <YCLOUD_API_BASE_URL>/balance
X-API-Key: <YCLOUD_API_KEY>
```

Synthetic response fixture:

```json
{"amount": 190.0765, "currency": "USD"}
```

Do not put a real key, account identifier, response, or customer financial data
in examples, fixtures, logs, or generated artifacts.

## Outcome requirements

Adapt the result to planning, implementation, execution, or evaluation, while making these
items explicit:

1. **Matched contract** — source hash, `GET /balance`, `balance-retrieve`, no
   request body, `200 Balance`, and required `amount`/`currency` fields.
2. **Interpretation** — numeric/precision strategy, ISO 4217 treatment, unknown
   property handling, freshness label, and every project-owned threshold or
   policy clearly separated from YCloud facts.
3. **Integration placement** — trusted-server adapter, placeholder
   Authentication handoff, mock transport seam, and redacted request-ID
   observability.
4. **Response and rate handling** — standard error envelope, no invented
   balance-specific statuses, defensive rate headers, `Retry-After`, and any
   bounded GET retry labeled as project policy.
5. **Tests** — no-parameter/no-body request construction; decimal and currency
   preservation; missing/wrong-typed fields; unknown fields/currency-like
   values; synthetic error/request-ID fixtures; `429` scheduling; timeout and
   bounded-retry policy; readiness handoff without a readiness claim; and proof
   that no network call occurs.
6. **CANNOT** — secret disclosure, out-of-scope financial data,
   mutations/top-ups/history, currency conversion, affordability or readiness
   guarantees, inferred SDK methods, invented quotas/errors, or unlabelled
   caching and threshold policy.
7. **Handoff** — provide Balance evidence with provenance and freshness to the
   Architect or project-owned readiness consumer, which owns the final policy
   decision. Return capability-row status, artifacts, tests/results, and
   unknowns; state when no handoff is needed.

## Safety and source priority

Use the pinned OpenAPI source first, generated references second, and this
workflow third. Keep provider contract, reviewed runtime facts, and project
policy visibly separate. Apply the shared execution policy to real operations
and keep local verification synthetic.
