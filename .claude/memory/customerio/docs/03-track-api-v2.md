# Track API v2 — identify, track, idempotency

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/messaging/objects-data/objects/create.md ·
https://docs.customer.io/integrations/api/track/tag/track-events/trackAnonymous ·
https://docs.customer.io/files/journeys-track.json

## Base URL (EU)

`https://track-eu.customer.io/api/v2/entity` — the entity endpoint used for both
`identify` and `track` actions in Track API v2. Auth is HTTP Basic: `site_id` as
username, `api_key` as password (`Authorization: Basic base64(site_id:api_key)`).

## The `identify` action

```json
{
  "type": "person",
  "identifiers": { "id": "<person_id>" },
  "action": "identify",
  "attributes": { "email": "...", "first_name": "...", "marketing_consent": true }
}
```

`type: "object"` is also supported for non-person entities (e.g. an order or account
object related to multiple people via `cio_relationships`) — useful when a design
needs an entity beyond the person record itself, not required for a simple
person-only identify flow.

## The `track` action

Same entity endpoint, `action: "event"` (not literally shown as `"track"` in the v2
entity shape — the SDKs expose a `.track()` method that constructs this under the
hood). Properties are freeform key/value.

## Idempotency

- **Track API v1** anonymous-event endpoint (`POST /api/v1/events`) accepts an optional
  `id` (a ULID) used to **deduplicate events**. If an event is submitted with an `id`
  that was already received, the duplicate is ignored. Note this is the **v1** anonymous
  endpoint's field name (`id`); server-side client libraries (Python, Node) expose the
  same concept as `message_id` on `.track()`, deduplicated within a **12-hour window**.
- If an application already has its own idempotency-key convention for outbound
  events, mapping that key onto whichever idempotency field the chosen SDK/endpoint
  exposes is the right pattern — and sending it additionally as an **event property**
  gives downstream visibility, since the platform-level dedup window (12h) is shorter
  than "forever" and a retry days later wouldn't be caught by the platform alone.
  **Verify the exact idempotency semantics of the v2 entity endpoint specifically
  before relying on it** — the docs excerpts available don't confirm the v2 entity
  endpoint (as opposed to the v1 events endpoint or an SDK's `message_id`) has the
  same 12-hour dedup window; this is the kind of claim worth a live verification
  rather than an assumption carried over from a different endpoint's documented
  behavior.

## Ordering: identify-before-track

A design that gates campaign entry on a consent (or similar) attribute typically
depends on `identify` being applied **before** a subsequent `track` is evaluated by a
campaign's entry filter. The docs do not make an explicit ordering guarantee for two
back-to-back API calls — this is worth a live-verified test rather than an assumed
fact, with a documented fallback if ordering can't be relied on (e.g. carrying the
gating attribute as an event property instead of a profile attribute).

## Practical implication

Don't hand-roll the v2 entity JSON shape from scratch in application code — the
official `customerio-node` SDK's `TrackClient.identify()` / `.track()` methods build
it (see `repos/sdks-and-tools.md`). Reach for the raw endpoint only if the SDK's shape
doesn't support something a specific design needs.
