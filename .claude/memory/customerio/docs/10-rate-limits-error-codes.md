# Rate limits & error codes

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/api/track · https://docs.customer.io/api/app ·
https://docs.customer.io/openapi.json · https://docs.customer.io/integrations/data-in/connections/classic-api/invalid-api-requests.md

## Rate limits

| API | Limit |
|---|---|
| Track API | 1000 requests/second. "Not strictly enforced" per the docs, but exceeding it consistently risks throttling or data loss — treat it as a real ceiling, not a soft target. |
| App API (general) | 10 requests/second for most endpoints |
| App API — transactional sends | Shares a **3000 requests / 3 seconds** soft limit with Track and Pipelines APIs (i.e. effectively ~1000 req/s, same order as Track) |
| App API — API-triggered broadcast | 1 request per 10 seconds — a much tighter limit; do not call this in a loop |

Most projects' send volumes will sit nowhere near these ceilings under normal
operation. The limit that could plausibly bite is the 10 req/s general App API limit
if a bulk backfill or export job polls too aggressively — throttle any scheduled
backfill job accordingly.

## Error shapes

- Track API validation error: `{"meta": {"error": "name cannot be blank"}}` — a flat
  `meta.error` string, not a structured field-level error array.
- App API / OpenAPI-documented endpoints use more conventional error responses with a
  `message` field per offending field, plus standard HTTP status semantics: `401`
  unauthorized, `429` rate limited, `500` internal server error (the docs specifically
  call out Liquid-render failures as a `500` case — relevant only if a template uses
  Liquid; an app-rendered raw-body send sidesteps this).

## Retry classification (confirmed via the official Node SDK, not the docs page)

The `customerio-node` SDK's built-in retry logic (see `repos/sdks-and-tools.md`)
classifies these as retryable: network errors (connection reset/refused, DNS, timeout)
and HTTP `408, 429, 500, 502, 503, 504, 522, 524`. Everything else (`400, 401, 404,
422`, etc.) is returned immediately, no retry. This is a good baseline for any
custom outcome-classification logic (`failed` vs `ambiguous` vs a retryable status) —
worth adopting the SDK's exact status-code list rather than re-deriving one from
scratch, since it's already tuned against real production traffic patterns.
