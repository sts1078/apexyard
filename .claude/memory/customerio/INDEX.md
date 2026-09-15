# Customer.io knowledge base — index

Curated, **generic, cross-project** corpus for the `customerio-specialist` agent. Every
file below is topic-scoped, carries `Source:`/`Fetched:` provenance, and is a
**curated summary with a practical implication called out** — not a documentation
mirror. It covers Customer.io platform mechanics and general practitioner know-how
that hold for any adopter; it does not, and must not, contain any specific managed
project's account facts, ticket/AgDR references, template keys, or team member names.

**This is the public layer.** Project-specific facts, decisions, and open questions
live in each managed project's own **private** layer — see "Where project-specific
facts live" below. On activation, the specialist agent reads this index first, then
loads only the topic files relevant to the question at hand, then resolves and reads
the active project's private layer if the question is project-specific.

## `docs/` — Customer.io platform documentation (15 files)

| File | Answers |
|---|---|
| `01-account-regions-eu.md` | How region works, EU vs US, why "GDPR compliant" ≠ "EU residency" |
| `02-workspaces.md` | What a workspace is, how staging/prod isolation works |
| `03-track-api-v2.md` | `identify`/`track`, the entity endpoint, idempotency, ordering guarantees |
| `04-app-api-transactional.md` | `POST /v1/send/email` — raw body, `send_to_unsubscribed`, `auto_create`, `transactional_message_id` as a metrics label |
| `05-reporting-webhooks.md` | Payload shape, `x-cio-signature` HMAC verification, backfill, at-least-once delivery |
| `06-subscriptions-topics-unsubscribe.md` | Global unsubscribe vs. topics, `List-Unsubscribe`, why an app-owned unsubscribe link is often the right call |
| `07-campaigns-automations.md` | Entry filters (a consent-gate mechanism), message status, trigger Liquid data, language variants |
| `08-segments.md` | Segment API surface, and the limit that segment *conditions* are UI-authored, not API-authored |
| `09-attributes-objects.md` | Person attributes, objects/relationships, Premium sensitive-data marking (internal visibility, not a legal control) |
| `10-rate-limits-error-codes.md` | Track/App API rate limits, error shapes, retryable status codes |
| `11-data-compliance-gdpr-dpa.md` | The Sensitive Data prohibition, HIPAA guidance that independently confirms the content-free-notification pattern, sub-processor gap |
| `12-mcp-server-and-cli.md` | MCP vs. the official CLI — and why Customer.io's own docs say a terminal coding agent should prefer the CLI |
| `13-ai-features-settings.md` | The account-level AI toggle, content filtering, what's worth verifying before enabling |
| `14-sending-domains-deliverability.md` | DKIM/SPF/DMARC per workspace, link tracking, RFC 8058 one-click compliance |
| `15-cdp-data-pipelines.md` | Track API vs. Pipelines/CDP API — and when each is the right choice |

## `repos/` — GitHub ecosystem (1 file)

| File | Answers |
|---|---|
| `sdks-and-tools.md` | Official `customerio` org repos, the case for building on `customerio-node` (EU region guard, built-in retry, webhook signature helper) instead of hand-rolled fetch calls, third-party clients evaluated and passed over |

## `practitioner/` — know-how beyond the official docs (8 files)

| File | Answers |
|---|---|
| `01-transactional-vs-campaign.md` | When to use each, and when it's worth departing from generic vendor advice (app-owned templates for a fallback-gated message class) |
| `02-unsubscribe-handling.md` | Why a vendor's suppression list is never the source of truth, the four-layer containment pattern |
| `03-eu-gdpr-considerations.md` | The generalized lesson from real ESP-evaluation history: "GDPR compliant" marketing claims vs. an actual DPO residency ruling; evidence discipline |
| `04-webhook-idempotency.md` | The ~7-day retry window, at-least-once delivery, signature verification is opt-in work |
| `05-two-workspace-environments.md` | Confirms the staging/prod-as-separate-workspaces pattern; the "not designed specifically for testing" caveat |
| `06-template-management-via-api.md` | The general trade-off behind app-owned vs. vendor-owned templates — why any gateway-fallback design needs app-owned rendering |
| `07-common-pitfalls.md` | **Highest-value single finding**: the millisecond-vs-second timestamp bug that silently corrupts date-based segment conditions |
| `08-migrating-from-klaviyo.md` | What generic migration-tool content offers (little) vs. platform-specific quirks worth knowing when migrating off Klaviyo |

## Where project-specific facts live

Every managed project that uses Customer.io has its own **private** layer at
`<projects_dir>/<project-name>/customerio/`, resolved via `portfolio_projects_dir`
(from `.claude/hooks/_lib-portfolio-paths.sh`) against the active project named in the
portfolio registry (`apexyard.projects.yaml`). That layer holds the project's own
account/workspace facts, decision records, open questions, and enforceable
project-specific handbook rules (the latter under that portfolio's
`custom-handbooks/general/` directory, which Rex reads as a second discovery layer
alongside the public `handbooks/` tree).

**Never write a project name, account ID, workspace ID, ticket number, AgDR number, or
team member name into this public corpus.** If a research finding is genuinely
project-specific, it belongs in that project's private layer, not here.

## Corpus discipline

- **Documentation-sourced claims are marked as such; live-verified claims are marked
  as such.** Never blur the two — see `practitioner/03-eu-gdpr-considerations.md` §
  "Evidence discipline."
- **This corpus is a snapshot dated 2026-09-15.** Customer.io's docs change. Re-fetch a
  `docs/` topic via Context7/WebFetch before relying on a claim that has consequences
  (a compliance claim, a rate limit near a design's actual volume, an SDK
  version-specific behavior) if it's been more than a few weeks since the `Fetched:`
  date.
- **24 files total** (15 docs + 1 repos + 8 practitioner + this index) — comfortably
  under the 150-file ceiling this corpus was asked to stay under, and deliberately
  smaller than an earlier draft that mixed in project-specific content now split out
  to the private layer.
