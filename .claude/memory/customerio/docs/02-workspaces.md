# Workspaces

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/integrations/api/app/tag/workspaces/listWorkspaces

## What a workspace is

A workspace is a sub-division of an **account**. All workspaces in an account share the
account's region (see `01-account-regions-eu.md`) but each workspace has its own:

- Track API credentials (site ID + API key)
- App API token
- People, campaigns, segments, sending domains, and suppression state
- Reporting-webhook signing key

This is a common mechanism for **staging/prod isolation** — a separate workspace per
environment, each with its own credentials, rather than polluting a single
workspace's event/metric names with an environment prefix (a string-prefixing scheme
is easy to forget in a hotfix, and pollutes production event names when it is).

## Listing workspaces via the App API

```bash
curl https://api.customer.io/v1/workspaces \
  --header 'Authorization: Bearer YOUR_SECRET_TOKEN'
```

Returns each workspace's `id`, `name`, `messages_sent`, `billable_messages_sent`,
`people`, `object_types`, `objects` — useful for a sanity dashboard, not for reading
each workspace's own credentials (a workspace's API keys are workspace-scoped and
retrieved from that workspace's own settings, not this endpoint).

## Practical implication for the specialist agent

If a managed project's design calls for environment-isolated workspaces (e.g.
staging/prod), each workspace needs its own secret (Track site ID, Track API key, App
API token, webhook signing key) stored per environment — never let one environment's
deploy hold another environment's credentials. Whether a given managed project's
workspace split has actually been created yet, and which workspace id(s) exist today,
is a **live, project-specific fact** — check that project's own private knowledge
layer (see this corpus's `INDEX.md`) before assuming a split exists or writing code
that references a specific workspace.
