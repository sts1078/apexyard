# Handbook: Customer.io integration

**Scope:** diffs touching a project's Customer.io client code, its webhook route, or
any code that constructs a Track API / App API request.
**Enforcement:** advisory

> This handbook lives under `handbooks/general/` because it's meant to apply
> wherever a managed project integrates with Customer.io, not to one specific
> project's diff shape. If a specific project has its own additional, sharper rules
> (exact template keys, exact allowlist conditions, exact AgDR citations), those live
> in that project's own private-portfolio custom handbook
> (`custom-handbooks/general/`) per `docs/multi-project.md`'s private-layer
> convention — Rex reads both layers together for that project's diffs. This file
> stays intentionally project-agnostic.

## The rule

1. **Every Customer.io endpoint, SDK client, or MCP config should target the region
   matching the project's account.** For an EU-residency-constrained project, that
   means `track-eu.customer.io`, `api-eu.customer.io`, `cdp-eu.customer.io`,
   `mcp-eu.customer.io` — never the bare/US-default host, and never
   `RegionUS`/an unset region on the official SDK. A US-host request on an
   EU-residency-constrained account risks logging data in the US even transiently.
2. **A message class a project has decided is app-owned (rendered from the
   project's own templates) should stay app-owned** — no Customer.io-hosted template
   for that class. `transactional_message_id` may be passed only as a metrics label
   (with `auto_create: true`), never as the actual content source, for such a class.
   This pattern is common for any message class with a fallback-gateway requirement
   (see `.claude/memory/customerio/practitioner/06-template-management-via-api.md`),
   since a vendor-rendered template can't be reproduced byte-for-byte on a fallback.
3. **No clinical, health, or otherwise sensitive identifier should appear in a
   Customer.io-bound payload** for a project that relies on content-free
   notifications to stay outside the vendor's Sensitive-Data definition — no
   diagnosis, medication name, dosage, or similarly specific detail in an event
   property, attribute, or message content. Only a neutral notification and a deep
   link to an authenticated surface.
4. **The transactional bypass (`send_to_unsubscribed`) should be set explicitly on
   every call where the bypass is actually intended**, never relied on as an assumed
   platform default.
5. **Every timestamp sent to Customer.io must be Unix seconds, not milliseconds.**
   The Track API accepts millisecond values without erroring and silently
   misinterprets them as seconds — a corruption that shows up only as a broken
   date-based segment/automation condition, with no error at send time.
6. **A retry loop around a Customer.io send should distinguish an ambiguous outcome
   (timeout, no response) from a retryable HTTP status**, and should not keep
   retrying an ambiguous outcome before falling back to any secondary gateway a
   project has for that message class. An unbounded retry-then-fallback design can
   produce more duplicate sends than intended during a vendor brownout.
7. **A webhook route must verify `x-cio-signature`** (HMAC-SHA256 of
   `v0:<timestamp>:<raw body>`) using the **raw request body bytes**, and must dedupe
   on the vendor's `event_id`, before writing to any downstream store. Prefer the
   official `customerio-node` `verifyRequestSignature` helper over a hand-rolled HMAC
   check.
8. **A project with its own signed unsubscribe-link design should never render
   Customer.io's own `{% unsubscribe_url %}` / subscription-center Liquid tags** on a
   message class that design is meant to cover — see
   `.claude/memory/customerio/docs/06-subscriptions-topics-unsubscribe.md`.

## Why

Rules 1, 3, and 8 encode compliance/consent patterns common to any
regulated-content adopter — a specific project's own reviewed decision record is the
authoritative version if it exists (check that project's private custom handbook).
Rule 5 is a documented platform-level footgun with no error signal at the call site
(see `.claude/memory/customerio/practitioner/07-common-pitfalls.md`) — exactly the
"returns success while doing something else" failure class the retired Mautic
specialist's handbook called "the API lies," now documented for a different vendor
before it costs debugging time here too. Rule 6 is what keeps a vendor brownout from
turning into a multi-copy notification storm. Rule 7 is standard webhook-security
hygiene, sharpened by the fact that Customer.io's own SDK does not verify signatures
for you automatically.

## What Rex flags

- A URL, `region:` value, or MCP config pointing at a host inconsistent with a
  project's own documented region requirement (check the project's private layer for
  what that requirement is).
- A create/send call to a Customer.io endpoint that supplies a message-class key
  a project has marked app-owned as the **content source**, rather than as a metrics
  label alongside a fully-rendered `subject`/`body`.
- A payload, event property, or attribute value that looks like a clinical or
  otherwise sensitive identifier on a message class meant to stay content-free.
- A `send_to_unsubscribed` field that's absent, or set via a variable that isn't
  provably intentional for the send in question.
- `Date.now()` (or another millisecond-precision value) passed directly as a
  Customer.io timestamp field without a `/1000` (or equivalent) conversion.
- A retry loop around a Customer.io send that doesn't distinguish an "ambiguous"
  (no-response) outcome from a retryable HTTP status, or that retries an ambiguous
  outcome before invoking any fallback-gateway arm.
- A webhook handler that reads an already-parsed body (rather than the raw bytes)
  when computing the HMAC, or that has no `event_id` uniqueness check before writing
  to its downstream store.
- A template containing `{% unsubscribe_url %}`, a link to Customer.io's hosted
  subscription center, or any Customer.io-native unsubscribe mechanism, on a message
  class the project's own design is meant to cover with its own link.

## Sample finding

> `nit:` `<client file>:42` — `new TrackClient(siteId, apiKey)` omits an explicit
> region. If this project's account is region-restricted, the SDK's own docs warn
> requests route through the US host first and may log data there transiently before
> redirecting. Add the correct explicit region.
>
> `suggestion:` `<client file>:88` — `timestamp: Date.now()` is milliseconds;
> Customer.io's Track API silently misinterprets a millisecond value as Unix seconds
> with no error, which will corrupt any date-based segment condition reading this
> attribute. Convert to seconds: `Math.floor(Date.now() / 1000)`.

## What's NOT a violation

- Marketing campaign content built in Customer.io's own editor — that's the intended,
  reviewed use of the vendor's UI for a project's marketing class. Rules 2–3 and 8
  apply to message classes a project has specifically decided must stay app-owned or
  content-free, not to marketing by default.
- A Customer.io-hosted template or subscription-center reference in **test/fixture
  code** clearly scoped to a non-deliverable test domain, exercising what the platform
  does — as long as it isn't wired into a real send path.
- Passing `transactional_message_id` alongside a fully-rendered `subject`/`body` with
  `auto_create: true` — this is the correct pattern (metrics label, not content
  source), not a violation of rule 2.

---

*Part of [ApexYard](https://github.com/me2resh/apexyard) — adopter handbook. The
specialist agent lives at `.claude/agents/customerio-specialist.md`. Project-specific
rules layer on top via each project's own private custom handbook.*
