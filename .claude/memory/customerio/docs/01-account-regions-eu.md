# Account regions — EU vs US

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/accounts/settings/data-centers.md · https://customer.io/pricing

## The core fact

Region is chosen **once, per account, at account creation** — not per workspace. Every
workspace in the account inherits the account's region. **Migrating between regions is
not supported.** If an account was created in the wrong region, the fix is a new
account, not a support ticket.

The EU data center is physically located in **Belgium**.

## How to verify which region an account is in

```bash
curl --request GET \
  --url https://track-eu.customer.io/api/v1/accounts/region \
  --header "Authorization: Basic $(echo -n site_id:api_key | base64)"
```

Returns `eu` or `us`. Run this against the EU host specifically — hitting the US host
with EU credentials will not reliably tell you the account's actual region.

## Compliance framing

Both US and EU data centers are described by Customer.io as GDPR-compliant, and
Customer.io is certified under the EU-US Data Privacy Framework (DPF) for
US↔EU transfers. That framing is the vendor's own — it does **not** automatically
substitute for a specific DPO ruling. A DPO's actual prerequisite for a
regulated/health-adjacent adopter is often **EU data residency** specifically, not
"any GDPR-compliant region" — the DPF-transfer path is exactly the mechanism a
US-only vendor relies on, and it's exactly what a residency-specific ruling rejects.
If your project's compliance posture requires EU residency, only the **EU region**
setting satisfies that; the US region's GDPR-compliance marketing claim on its own is
not equivalent.

## What the docs do NOT enumerate

Customer.io's docs do not spell out whether **backups, logs, and support access**
stay region-scoped, only the primary data store. This is exactly the kind of claim
that needs the vendor's written confirmation rather than being inferred from the
region setting. Ask explicitly: does a support engineer accessing an EU workspace do
so from an EU jurisdiction, and are backups replicated outside the EU?

## Pricing / tier note

The public pricing page lists the EU data center as available on every tier ("customers
can choose … US or EU"), including Essentials (5,000 profiles, 1M emails/month
included). Additional workspaces are a paid add-on — relevant to any multi-workspace
(e.g. staging/prod) setup, since that multiplies the workspace add-on cost.

## Practical implication for the specialist agent

Any code, script, or curl example this agent writes **must** target the `-eu` host
variants (`track-eu.customer.io`, `api-eu.customer.io`, `cdp-eu.customer.io`,
`mcp-eu.customer.io`) whenever the managed project's account is EU-region — never the
bare (US-default) hosts. **Which region a specific managed project's account is in,
and any confirmed live account facts, live in that project's own private knowledge
layer** (see this corpus's `INDEX.md` for how to resolve it) — never assume a region
or hardcode an account/workspace identifier here in the shared, cross-project layer.
