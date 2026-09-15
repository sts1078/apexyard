# Customer.io GitHub repos — SDKs, CLI, and what's worth using

**Fetched:** 2026-09-15
**Source:** `gh search repos`, `gh api orgs/customerio/repos`, `gh api repos/customerio/customerio-node/readme`

Not cloned into this repo — linked and summarized only, per the standing instruction
not to vendor third-party source into the ops fork.

## Official `customerio` org — actively maintained, relevant

| Repo | Language | Last push (as of fetch) | Notes |
|---|---|---|---|
| [`customerio/customerio-node`](https://github.com/customerio/customerio-node) | TypeScript | 2026-09-13 | **The one to use for a TypeScript project.** See § below — TypeScript-native, EU region built in, built-in retry, `verifyRequestSignature` helper. |
| [`customerio/cli`](https://github.com/customerio/cli) | — | current | The official CLI (`npm i -g @customerio/cli` or `go install github.com/customerio/cli@latest`). Customer.io's own docs recommend this over MCP for a terminal coding agent — see `docs/12-mcp-server-and-cli.md`. |
| [`customerio/go-customerio`](https://github.com/customerio/go-customerio) | Go | 2026-07-29 | Official Go client. Relevant only to a Go-stack project. |
| [`customerio/customerio-python`](https://github.com/customerio/customerio-python) | Python | 2026-07-24 | Official Python client. Relevant only to a Python-stack project. |
| [`customerio/customerio-ruby`](https://github.com/customerio/customerio-ruby) | Ruby | 2026-07-24 | Official Ruby client. Relevant only to a Ruby-stack project. |
| [`customerio/customerio-ios`](https://github.com/customerio/customerio-ios), [`customerio-android`](https://github.com/customerio/customerio-android), [`customerio-reactnative`](https://github.com/customerio/customerio-reactnative), [`customerio-expo-plugin`](https://github.com/customerio/customerio-expo-plugin) | Swift / Kotlin / TS / TS | active | Mobile push/in-app SDKs — relevant if a project's scope extends to mobile push on the same profile. |
| [`customerio/action-destinations`](https://github.com/customerio/action-destinations) | TypeScript | 2026-09-09 | Segment-ecosystem destination action definitions; relevant only to a project routing through Segment. |
| [`customerio/liquid`](https://github.com/customerio/liquid) | Ruby | 2021 (stale) | The Liquid template-language implementation Customer.io's own editor uses. Reference only if debugging a marketing-team-authored Liquid template; irrelevant to an app-owned raw-body send path (no Liquid involved). |
| [`customerio/premailer`](https://github.com/customerio/premailer) | Ruby | active | HTML-email CSS-inlining preflight tool. Marginally relevant if a project's own email templates already use inline styles by convention (in which case the CSS-class-inlining problem this tool solves mostly doesn't arise). |
| [`customerio/examples`](https://github.com/customerio/examples) | Go | 2021 (stale) | Official examples repo — stale, Go-only. Low relevance. |

**No official Terraform provider exists** (`gh search repos "customerio terraform"`
returned zero results from the `customerio` org). Any Customer.io infra-as-code has to
be hand-rolled — workspace and domain setup is largely a one-time UI/API task rather
than something worth Terraform-provider tooling for at typical adopter scale.

## `customerio-node` — the SDK worth building on for a TypeScript project

This is the single most load-bearing research finding of this survey. Full details
in `docs/03-track-api-v2.md`, `docs/04-app-api-transactional.md`,
`docs/05-reporting-webhooks.md`, and `docs/10-rate-limits-error-codes.md` — summarized
here:

- **Built on standard `fetch`** since v5.0.0 — runs on Node (current/LTS/maintenance),
  Bun (CI-tested), and other fetch-compatible runtimes (Deno, Cloudflare Workers —
  untested but likely fine).
- **Three separate clients**: `TrackClient` (identify/track), `APIClient` (App API —
  transactional sends, exports, campaigns), `PipelinesClient` (CDP — not needed by a
  project using Track API v2 as its own event store's downstream executor, see
  `docs/15-cdp-data-pipelines.md`).
- **`RegionEU` built in**: `new TrackClient(siteId, apiKey, { region: RegionEU })`.
  Critically, the SDK's own docs warn that **if you don't specify a region and your
  account is EU, requests still route to the US host first and get redirected** — "this
  may cause data to be logged in the US." **Always pass the correct region explicitly
  for any EU-residency-constrained account; never rely on the SDK's auto-redirect.**
- **Built-in retry with exponential backoff + jitter**, tuned against real traffic:
  retries network errors and HTTP `408/429/500/502/503/504/522/524`; honors
  `Retry-After`; configurable `maxRetries`, `minTimeoutMs`, `maxTimeoutMs`,
  `maxTotalBackoffMs`. Retries are safe to replay (same request body reused).
- **`verifyRequestSignature` webhook helper** ships in `docs/webhooks.md` inside the
  repo — replaces hand-rolling the `v0:timestamp:body` HMAC-SHA256 check.

**Trade-off worth recording explicitly**: a hand-rolled ESP-client interface with its
own retry/backoff (often lifted verbatim from a previous vendor's client logic) is a
common pattern when migrating between ESPs. Given `customerio-node` already ships a
production-tuned retry policy, a region guard, and a webhook-signature verifier, **the
stronger design choice is usually to wrap `customerio-node`'s `TrackClient` +
`APIClient` inside whatever internal client interface a project already has**, rather
than hand-rolling HTTP calls against the raw endpoints and re-implementing
retry/backoff from scratch. This reduces the amount of custom code a security
reviewer has to review, and inherits a retry policy already exercised against
production traffic by the vendor's own userbase. The one place raw `fetch` might
still make sense is a raw-body transactional-send path, if the SDK's `APIClient`
doesn't cleanly expose fields like `send_to_unsubscribed` / `tracked` /
`disable_css_preprocessing` as first-class options — verify the SDK's actual current
method signature before deciding either way; this is a build-time verification, not
something to assume from this summary alone.

## Third-party / community — evaluated, not generally worth reaching for

| Repo | Notes |
|---|---|
| `omaihq/customerio-cli` (Go) | Unofficial CLI. No reason to use over the official `customerio/cli`. |
| `casetext/customerio-client` (JS), `jacobemerick/customerio-client` (PHP), `serokell/customerio-client` (Haskell) | Unofficial clients in various languages. Not a fit for a TypeScript project — the official `customerio-node` is actively maintained (pushed within the last two days as of this fetch), so there's no gap for a third-party client to fill. |
| `LimeJourney/limeJourney`, `laudspeaker/laudspeaker` | Open-source Customer.io **alternatives** (self-hosted engagement platforms), surfaced by a generic "customer.io" search because of their positioning copy. Not tools *for* Customer.io — noted only so a future search doesn't waste time re-discovering they're unrelated. |
| `digitalcreations/CustomerIOSharp`, `UserScape/php-customerio`, `printu/customerio` | Older/unofficial .NET and PHP clients. Only relevant to a .NET or PHP stack. |

## Note on unrelated same-name collisions

A generic `customer.io` / `customerio` search surfaces several **unrelated** repos
that happen to share the string in their name or description (`tywo45/t-io` — a Java
network framework; `RotherOSS/otobo` — a ticketing system that mentions "Customer
Service"). These are noise, not signal — recorded here only so the specialist agent
doesn't re-investigate them on a future search.
