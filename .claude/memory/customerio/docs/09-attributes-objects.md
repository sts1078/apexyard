# Attributes & objects

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/messaging/objects-data/objects/create.md · https://docs.customer.io/openapi.json · https://docs.customer.io/release-notes

## Person attributes

Set via `identify` (Track API v2 entity endpoint, or the v1 `PUT /api/v1/customers/{id}`
shape older integrations use). A minimal, deliberately-starved attribute set (email,
name, locale, consent flag, and nothing else — no clinical/sensitive data) is a common
and defensible pattern for any project keeping Customer.io as a downstream executor
rather than the customer-data system of record.

## Objects (non-person entities)

Track API v2's `type: "object"` action can create an object (e.g. `object_type_id` +
`object_id`) and relate it to one or more people via `cio_relationships`. Useful when
a design needs an entity independent of any one person's own lifecycle (e.g. a
"subscription" or "order" object).

## Sensitive-data attribute marking (Premium plan)

Premium-plan accounts can mark specific attributes as **sensitive** in the Data Index,
restricting visibility to specific team roles (Admins / Workspace Admins only, by
default). This is an **internal team-visibility control**, not a data-processing /
legal-basis control — do not confuse it with a vendor's contractual "Sensitive Data"
definition in its DPA (health data, financial account numbers, etc. — see
`docs/11-data-compliance-gdpr-dpa.md`). Marking a consent attribute or similar as
UI-sensitive could be a reasonable internal-hygiene step but has zero bearing on
whether content sent through the platform violates a DPA's Sensitive Data prohibition.

## AI-assisted attribute description

Release notes mention AI-generated descriptions for attributes/events in the Data
Index, intended to improve AI-assisted segment-building suggestions. Optional,
cosmetic — see `docs/13-ai-features-settings.md` for how to keep AI features scoped
appropriately and disabled where not wanted.
