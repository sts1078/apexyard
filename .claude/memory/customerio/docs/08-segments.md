# Segments

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/integrations/api/app · https://docs.customer.io/get-started/segments-and-profiles-data.md

## Two kinds

- **Data-driven segments** — update automatically as profile attributes/events change.
  This is the kind relevant to a consent-gated campaign design: a marketing campaign's
  entry condition (`marketing_consent = true`) is effectively a data-driven segment
  membership question.
- **Manual segments** — static lists (CSV upload, workflow action). Relevant only to
  designs that need bulk static recipient lists, not to an event-driven transactional
  design.

## API surface

```
GET    /v1/segments                       list segments
POST   /v1/segments                       create a manual segment (name + description only — data-driven segment conditions are UI-authored, not API-authored)
GET    /v1/segments/{id}                  get one segment
DELETE /v1/segments/{id}                  delete a manual segment
GET    /v1/segments/{id}/dependencies     what depends on this segment
GET    /v1/segments/{id}/customer_count   count
GET    /v1/segments/{id}/membership       list members
```

Filtering people by segment + attribute combined:

```bash
curl --request POST \
  --url 'https://api.customer.io/v1/customers?start=...&limit=...' \
  --header 'Authorization: Bearer ...' \
  --data '{"filter":{"and": [{"segment":{"id": 4}},{"attribute":{"field": "likes_pizza","operator": "eq","value": true}}]}}'
```

## Practical implication

**The API does not create data-driven segment conditions** (only manual-segment
metadata). A data-driven segment's rule logic is authored in the Customer.io UI. This
matters for the specialist agent's build discipline: unlike a REST-scriptable
platform where every object can be built as code, Customer.io's segment/campaign
entry-condition authoring is UI-native for anything beyond the raw attribute the app
already writes (e.g. a consent flag). The agent can draft the condition logic and hand
it to whoever owns the marketing-campaign build (the design owner) to enter in the
builder — it should not claim it can script a data-driven segment's rule tree via the
API, because it can't.
