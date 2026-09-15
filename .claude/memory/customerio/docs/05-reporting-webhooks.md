# Reporting webhooks — events back, signature verification

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/integrations/api/webhooks ·
https://docs.customer.io/integrations/data-out/connections/webhooks.md ·
https://docs.customer.io/integrations/data-out/connections/reporting-webhook ·
https://docs.customer.io/webhooks

## Payload shape

```json
{
  "metric": "subscribed",
  "event_id": "01E4C4CT6YDC7Y5M7FE1GWWPQJ",
  "object_type": "customer",
  "timestamp": 1613063089,
  "data": { "customer_id": "...", "identifiers": { "id": "...", "email": "...", "cio_id": "..." } }
}
```

Events are organized by `object_type` (`customer`, `email`, `push`, `sms`, …) and
`metric` (the specific event). Email metrics referenced across the docs excerpts
include `email_sent`, `email_delivered`, `email_opened`, `email_clicked`,
`email_bounced`, `email_dropped`, `email_spammed`, `email_failed`,
`email_undeliverable`, `email_unsubscribed`. Confirm the exact metric name set against
the live webhook config UI before wiring a route — the docs excerpts don't enumerate
every metric name in one place.

`event_id` is the field to dedupe on — a natural unique key for a
`(source, external_id)` uniqueness constraint on whatever table ingests these events.

## Signature verification

Headers: `x-cio-timestamp` (unix seconds) and `x-cio-signature` (hex-encoded
HMAC-SHA256).

The signed message is **`v0:<timestamp>:<raw request body>`**, HMAC-SHA256'd with the
workspace's webhook signing key:

```go
mac := hmac.New(sha256.New, []byte(WebhookSigningSecret))
mac.Write([]byte("v0:" + strconv.Itoa(XCIOTimestamp) + ":"))
mac.Write(RequestBody) // the RAW body bytes, not a re-serialized/parsed version
computed := mac.Sum(nil)
hmac.Equal(computed, signature) // constant-time compare
```

Two things worth flagging for any webhook-route implementation:

1. **Use the raw bytes**, not a JSON.parse → JSON.stringify round-trip — key ordering
   or whitespace differences will break the signature. Frameworks that parse the body
   before your handler sees it (e.g. certain Next.js body-parsing defaults) need a raw
   body capture.
2. **The official `customerio-node` SDK ships a `verifyRequestSignature` helper**
   (see `repos/sdks-and-tools.md`) — reach for that instead of hand-rolling the HMAC.

## Timestamp replay window

A reasonable defensive practice is rejecting a webhook whose timestamp is more than a
few minutes old — but confirm Customer.io doesn't itself legitimately retry webhooks
with an older timestamp under normal operation before picking a specific window; an
over-tight window would silently drop a real, late-but-legitimate delivery event.

## Backfill (missed webhook recovery)

`GET /v1/transactional/{id}/deliveries` and the equivalent campaign-messages endpoint
(`GET /v1/campaigns/{campaign_id}/messages`) return delivery records with metrics and
timestamps — a scheduled backfill job can poll these to catch anything a webhook
outage missed. See `docs/07-campaigns-automations.md` for the campaign-messages
endpoint shape.

## At-least-once delivery — expect duplicates

Multiple independent sources (see `practitioner/04-webhook-idempotency.md`) describe
Customer.io's webhook delivery as retrying over up to a 7-day window on failure, which
means **the same event can arrive more than once**. An `event_id` uniqueness
constraint on the ingesting table is the correct defense — don't add a second, weaker
dedup mechanism on top.
