# Practitioner know-how — running staging + prod as separate workspaces

**Fetched:** 2026-09-15 (web search + vendor docs cross-reference)
**Sources:** https://docs.customer.io/accounts/workspaces/overview/ + general web search, listed below.

## The pattern, and one caveat worth flagging

Using **two Customer.io workspaces** (staging, prod) for environment isolation is a
standard pattern, and it's a strictly better mechanism than string-prefixing metric
names in a single workspace (which pollutes production event names and is easy to
forget in a hotfix). Practitioner sources confirm the pattern: "Workspaces can be used
as a sandbox to set up testing/staging environments... Each workspace is assigned its
own set of API keys and [is] completely separate from your other workspaces."

**The caveat, stated directly by the sources**: workspaces are "**not designed
specifically for testing**" — meaning Customer.io doesn't offer testing-specific
affordances (like a sandbox mode with relaxed rate limits, or a "this is not a real
send" flag) beyond the fact that a separate workspace's sends genuinely don't touch
production data. Treat a staging workspace as a **fully real, separate account** for
billing, deliverability reputation, and domain-authentication purposes — not a free
test mode. A recipient-allowlist guard at the application layer stays necessary even
on a staging workspace — the workspace boundary alone doesn't prevent an accidental
send to a real address if the test data itself contains one.

## Credential management pattern

Each workspace's Site ID, Track API Key, and App API Key should be managed through
environment-specific secrets, never shared or reused across workspaces. This is the
confirmed right shape for any project doing a staging/prod split.

## Promoting a campaign from staging to prod

Sources describe two viable patterns:

1. **Manually re-create** the campaign/automation in the prod workspace once validated
   in staging (campaigns aren't automatically portable between workspaces).
2. Some accounts support **copying workflow actions** between workspaces — check
   current account-tier support for this before assuming it's available.

This promotion concern applies only to campaigns built in the vendor's own editor —
for a design where app-decided sends are raw-body API calls (no vendor-side campaign
object at all — see `docs/04-app-api-transactional.md`), there's nothing to promote
between workspaces for those message classes.

## Sources

- [Workspaces in Customer.io](https://docs.customer.io/accounts/workspaces/overview/)
