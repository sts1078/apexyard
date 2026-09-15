# Subscriptions, topics, List-Unsubscribe, subscription center

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/messaging/channels/subscriptions/unsubscribe-faqs.md ·
https://docs.customer.io/integrations/api/app/tag/subscription-center/getTopics ·
https://docs.customer.io/integrations/api/app/tag/customers/getPersonSubscriptionPreferences ·
https://docs.customer.io/messaging/liquid/tag-list

## Global unsubscribe vs topics

Customer.io supports two suppression granularities:

- **Global unsubscribe** — a person's `unsubscribed` attribute; clicking a rendered
  `List-Unsubscribe` header link (RFC 8058 one-click) **always unsubscribes the person
  globally**, not just from the topic/campaign that sent the triggering message. This
  is confirmed directly by the docs: "clicking this header results in a global
  unsubscribe from all messages rather than a specific subscription topic."
- **Subscription topics** — finer-grained opt-in/out per topic, surfaced through a
  hosted **subscription center** (`{% manage_subscription_preferences_url %}` Liquid
  tag) and the App API (`GET /v1/subscription_topics`,
  `GET /v1/customers/{id}/subscription_preferences`).

## Why this matters for a purpose-granular unsubscribe design

Many designs that need to distinguish "opted out of marketing" from "still wants
transactional/clinical mail" render their **own** signed unsubscribe link, rather than
Customer.io's own `{% unsubscribe_url %}` or subscription center. This finding
explains why that's often the right call, not just a preference:

1. Customer.io's own one-click unsubscribe is **global by design** — it cannot express
   "unsubscribe from marketing but keep other message classes" on its own. A
   multi-layer containment design (own link, entry-filter projection, push
   withdrawals into the vendor, ingest the vendor's own webhook) exists precisely
   because a vendor-native unsubscribe collapses purpose granularity.
2. **Ad hoc test emails sent from the template composer do not include the
   `List-Unsubscribe` header** — worth knowing when manually testing a template in the
   Customer.io UI: the absence of the header on a composer test send does not tell you
   whether a real campaign send would include it.

## The header pair — `List-Unsubscribe` + `List-Unsubscribe-Post`

Worth verifying live: that a **campaign** send carries `List-Unsubscribe` +
`List-Unsubscribe-Post` pointing at whatever unsubscribe endpoint the application
owns (not Customer.io's), and that a **transactional** send carries **no**
`List-Unsubscribe` header at all. If Customer.io forces the header on every send
regardless of channel, a reasonable fallback is: record an unsubscribe from a
transactional/clinical send as a **marketing-only** withdrawal (the safe direction —
the other message class keeps delivering via its own per-call bypass, only marketing
consent is affected).

## Practical implication for the specialist agent

Don't point a template's unsubscribe link at Customer.io's own `{% unsubscribe_url %}`
/ subscription-center Liquid tags when the managed project has its own consent ledger
— and don't suggest the vendor's subscription-topics feature as the system of record
for consent. Customer.io should generally receive a consent attribute as a read-only
projection; it is not the system of record. Whether a specific managed project's
consent model works this way, and the exact unsubscribe-containment design in place,
is project-specific — check that project's private knowledge layer.
