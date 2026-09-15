# Practitioner know-how — webhook idempotency, production pitfalls

**Fetched:** 2026-09-15 (web search)
**Sources:** listed at the bottom, dated 2026 unless noted; general webhook-engineering
guidance plus Customer.io-specific notes where found.

## Customer.io-specific findings from practitioner sources

- **Retry window**: independent sources describe Customer.io retrying failed webhook
  deliveries over **up to a 7-day window**. This means a webhook consumer will
  plausibly see the same `event_id` **days apart**, not just within a short retry
  burst — a naive in-memory or short-TTL dedup cache would let a late duplicate
  through. A durable, permanent uniqueness constraint on the event id is the correct
  shape for a multi-day retry window.
- **At-least-once, not exactly-once**: Customer.io (like essentially every webhook
  provider) favors not losing events over strict exactly-once delivery. Treat every
  webhook handler as needing idempotent writes, unconditionally — never as an edge
  case to handle "if it happens."
- **Signature verification is opt-in work, not automatic**: the official
  `customerio-node` package does not verify signatures for you unless you call its
  `verifyRequestSignature` helper explicitly — it's on the integrator to wire it into
  the request-handling path. (Confirmed directly in the repo's own docs — see
  `repos/sdks-and-tools.md`.)

## General webhook-engineering lessons, applicable here

- **Dedupe on the provider's event ID, not on your own derived key.**
- **Respond fast, process async for anything heavier than a single insert.** Most
  providers time out retries after 10-30 seconds, and synchronous slow processing is a
  common cause of duplicate-triggering retries — keeping the synchronous handler path
  to one insert and pushing heavier work to a scheduled job is the safe default.
- **Event ordering is a separate problem from deduplication** — handling duplicates
  safely does not guarantee events arrive in the order they occurred. A design that
  treats each webhook event as an independent fact appended to a log (rather than
  depending on sequence) sidesteps this concern entirely.

## Sources

- [Guide to Customer.io Webhooks: Features and Best Practices](https://hookdeck.com/webhooks/platforms/guide-to-customerio-webhooks-features-and-best-practices)
- [Webhook idempotency: how to handle duplicate deliveries safely](https://dev.to/adal-cloud/webhook-idempotency-how-to-handle-duplicate-deliveries-safely-1m4)
- [Webhook Reliability 2026: Idempotency & Retry Reference](https://www.digitalapplied.com/blog/webhook-reliability-idempotency-retries-engineering-reference-2026)
- [Webhook Best Practices: Idempotency and Event Ordering](https://boldsign.com/blogs/webhook-best-practices-retries-idempotency/)

These are third-party practitioner sources, not Customer.io's own documentation —
treat the specific "7-day retry window" figure as a practitioner-reported claim worth
a quick live cross-check (e.g. Customer.io's own webhook settings page) rather than a
vendor-confirmed guarantee.
