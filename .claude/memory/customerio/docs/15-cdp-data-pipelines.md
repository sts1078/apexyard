# CDP / Data Pipelines — and when Track API v2 is the right choice instead

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/integrations/api/track-vs-cdp-api.md ·
https://docs.customer.io/integrations/api/cdp/email · https://docs.customer.io/journeys/channels/push/developer-guide

## Two distinct APIs, easy to conflate

| | Track API (Journeys) | Pipelines API (CDP / Data Pipelines) |
|---|---|---|
| Base URL (EU) | `track-eu.customer.io` | `cdp-eu.customer.io` |
| Timestamps | Unix integer | ISO 8601 string |
| Event shape | `data` object, direct key/value | `properties` (event) / `traits` (profile) objects |
| Positioning | Legacy, purpose-built, Journeys-native | Newer, broader feature set (geolocation, more integrations); Customer.io's own docs recommend it "for most integrations... especially for those not using a third-party CDP" |

**Customer.io's own docs now recommend the Pipelines API as the default entry point
for new integrations** — worth stating precisely, because it could read as "any new
migration should target CDP, not Track v2."

**Why Track v2 (not CDP/Pipelines) is still the right call for a project that already
owns its own customer event store.** If a project's architecture already treats its
own database as the source-of-truth event store, with Customer.io as a downstream
**executor** rather than the CDP, Track API v2's `identify`/`track` shape maps
directly onto a small, fixed event contract with the smallest possible change
surface. Adopting the Pipelines API in that case would mean re-deriving the same
contract in a different shape for no architectural gain, when Customer.io isn't
being used as the actual CDP.

## When CDP/Pipelines *would* become relevant

If a project later adopts Customer.io (or another CDP) as an actual customer-data
hub — ingesting browser/product events, not just a handful of lifecycle events —
Pipelines is the API built for that, and it's worth a fresh architectural decision at
that point rather than retrofitting it into an existing transactional/campaign-focused
design. Until then, don't steer a project toward Pipelines/CDP without a deliberate
reason — check whether a specific managed project has already made this call (and
why) in its private knowledge layer before assuming either direction.
