# Practitioner know-how — transactional vs. campaign, when to use each

**Fetched:** 2026-09-15 (web search, vendor + third-party sources)
**Sources:** https://docs.customer.io/journeys/send/transactional/campaign/ (vendor) ·
general web search results, dated and separated from vendor material below.

## Vendor guidance (docs.customer.io)

- Transactional messages are ones your audience expects **even if they've opted out of
  marketing** — receipts, password resets, shipping updates. Customer.io's own
  recommendation: use the Transactional API for these.
- Transactional-API sends **skip the campaign/journey engine entirely**, so they
  process faster and use less platform overhead than routing the same message through
  a campaign.
- If a transactional message needs a channel beyond email/SMS/push/in-app/inbox, or
  needs to trigger a downstream attribute change as a side effect, that's the signal to
  use a **transactional campaign** (an event-triggered campaign built to behave like a
  transactional send) instead of the plain Transactional API.

## A common message-class mapping

| Class | Vendor mechanism | Why |
|---|---|---|
| Sensitive/clinical/health notification | Transactional API (`POST /v1/send/email`), raw body | App decides every send; no campaign entry logic needed; needs the bypass flag set explicitly per call |
| Transactional/billing | Transactional API, raw body | Same reasoning |
| Marketing | Campaign / automation | Needs entry filters (consent gating), delays, branching — the campaign engine's actual job |
| Auth (password reset, 2FA) | Often kept off a third-party ESP entirely | Login-recovery latency and availability are frequently treated as security properties independent of any ESP |

This mapping is idiomatic to the platform, not an unusual use of it — transactional
and campaign each fit the vendor's own definition of the territory they're meant for.

## A vendor recommendation worth being deliberate about departing from

One piece of general vendor best-practice guidance: **"keep the content (HTML/Text)
inside the Customer.io UI, so your marketing team can fix a typo without a developer
deploying code."** This is good generic advice for a typical Customer.io customer —
and the opposite of what a design needs when a message class requires a **fallback
gateway** (e.g. falling back to a direct SES send if Customer.io is down). The reason:
two copies of the same template (vendor-held + app-rendered-elsewhere) drift, and a
fallback to a different gateway can't reproduce vendor-rendered content
byte-for-byte. When a message class has a hard fallback requirement, app-owned
rendering (render once, hand every gateway the same finished bytes) is close to a
hard requirement, not a stylistic choice — see
`practitioner/06-template-management-via-api.md`.

## Sources

- [Campaigns with transactional messages](https://docs.customer.io/journeys/send/transactional/campaign/)
- [Transactional Campaigns docs](https://www.customer.io/docs/journeys/transactional-campaign/)
