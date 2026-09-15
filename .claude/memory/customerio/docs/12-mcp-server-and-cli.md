# MCP server & CLI

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/ai/mcp/ide.md · https://docs.customer.io/ai/mcp/get-started · https://docs.customer.io/ai/cli/get-started

## Two tools, different fit for a terminal agent

- **MCP server** — `https://mcp.customer.io/mcp` (US) / `https://mcp-eu.customer.io/mcp`
  (EU) — configured in an IDE's or agent's MCP settings as an HTTP MCP server,
  OAuth-authenticated. Always use the host matching the target account's region.
- **CLI** — `npm install -g @customerio/cli` (also installable via
  `go install github.com/customerio/cli@latest`; public source at
  `github.com/customerio/cli`, see `repos/sdks-and-tools.md`). Notably, **Customer.io's
  own docs recommend the CLI over the MCP server specifically for terminal-based
  coding agents**: *"For terminal-based agents like Claude Code, the Customer.io CLI is
  recommended over the MCP server. The CLI provides direct command-line access to the
  full API surface without requiring the additional setup associated with the Model
  Context Protocol."*

## Practical implication for the specialist agent

This is directly relevant to how this agent should operate day to day:

- **Default to the CLI (or direct region-appropriate host API calls) for scripted,
  repeatable actions** — creating/reading campaigns, checking delivery status, running
  exports. This matches the vendor's own stated guidance for a coding agent's context,
  and the general discipline of scripting everything rather than leaving state that
  only exists as UI clicks.
- **Use the MCP server** when a session already has it configured and wants this agent
  to work through the same MCP tools interactively (e.g. to respect account-level
  permission scoping already set up via OAuth) — but don't assume MCP is the *default*
  path; the vendor's own docs say otherwise for exactly this agent's use case.
- Whichever path is used, it authenticates against the **same workspace
  credentials** — there's no separate credential model for MCP vs CLI vs raw API.

Whether MCP is actually enabled on a specific managed project's account, and which
region's endpoint it's configured against, is a project-specific fact — check that
project's private knowledge layer.

## Generic IDE config shape (for reference)

```json
{ "mcpServers": { "customerio": { "url": "https://mcp-eu.customer.io/mcp" } } }
```

Substitute the region-appropriate host into whatever MCP client config format the
harness expects; `docs.customer.io/ai/mcp/ide.md` shows the shape for several IDEs.
