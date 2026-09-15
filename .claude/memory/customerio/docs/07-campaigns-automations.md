# Campaigns & automations — triggers, entry filters, delays, message status

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/messaging/send/automations/triggers-and-filters.md ·
https://docs.customer.io/integrations/api/app/tag/campaigns/getCampaignMessages ·
https://docs.customer.io/integrations/api/app/tag/campaigns/campaignJourneyMetrics ·
https://docs.customer.io/liquid-tag-list

## Entry filters — the mechanism behind a consent-gated campaign

**"For all other automation types [not legacy segment-triggered], profiles will not
enter the journey if they do not match the filter after a 30-minute pause."** This
matters for any design that gates campaign entry on a consent attribute (e.g. an entry
filter of `marketing_consent = true`): a profile whose attribute is absent (never
matches `true`) is **dropped, never sent to** — confirmed as the platform's actual
behavior for non-legacy automations, not just an assumption. Two things to verify live
before trusting this fully:

1. Which automation type a specific campaign actually is (a Journeys "campaign" vs a
   "legacy segment-triggered automation") — the exemption clause above only applies to
   non-legacy types.
2. The **30-minute pause** before the filter re-check — a `track` immediately followed
   by an entry check might race against a slightly-delayed `identify`, though a design
   with a strict "identify before track" call ordering should make this moot in
   practice.

## Message status and "why not sent"

`GET /v1/campaigns/{id}/journey_metrics` returns aggregate counts (`started`,
`activated`, `exited_early`, `finished`, `converted`, `never_activated`, `messaged`)
per time bucket. `GET /v1/campaigns/{id}/messages` returns per-delivery records with a
`metric` filter (`attempted`, `sent`, `delivered`, `opened`, `clicked`) and a `drafts`
flag to see queued-but-unsent messages. Neither endpoint excerpt gathered here gives an
explicit enumerated list of "not sent" reason codes (e.g. "unsubscribed", "suppressed",
"filter not matched") — when diagnosing a campaign that isn't sending, check the UI's
per-person journey view first; it's more informative than the API for this specific
question.

## Trigger data in messages (Liquid)

```liquid
{{trigger.<data.property>}}              {# transactional / webhook-triggered / API-triggered broadcast #}
{{trigger.<object_type>.<attribute_name>}} {# object-triggered campaign #}
{{trigger.relationship.<attribute_name>}}  {# object/relationship-triggered campaign #}
```

Always pair with a fallback filter (`{{trigger.first_name | default: "there"}}`) —
any hand-built marketing campaign in the vendor's own editor should follow this
defensive-default discipline. A design that renders app-owned raw HTML instead of
using Liquid at all sidesteps this entirely for those message classes.

## API-triggered broadcasts

`POST /v1/api/campaigns/{campaign_id}/triggers` fires a broadcast campaign
programmatically — useful for a one-off marketing blast outside the normal
event-driven flow, distinct from triggering via a Track API event.

## Language variants

`PUT /v1/campaigns/{campaign_id}/actions/{action_id}/language/{language}` updates a
translated variant of a campaign action. The App API "now supports retrieving and
updating language variants for automations, newsletters, API-triggered broadcasts, and
transactional messages" — relevant to any project with existing translation-selection
logic for campaigns built in the vendor's own editor, though app-rendered
transactional/clinical sends would keep using their own translation pipeline
regardless.
