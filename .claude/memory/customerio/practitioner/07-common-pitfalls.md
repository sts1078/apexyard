# Practitioner know-how — common integration pitfalls

**Fetched:** 2026-09-15 (web search)
**Sources:** listed at the bottom. Third-party practitioner sources — verify anything
load-bearing against the live workspace before relying on it, same discipline as any
documentation-vs-live-verified split.

## 1. Two authentication systems, not interchangeable

**Track API uses HTTP Basic Auth** (`site_id` as username, `api_key` as password).
**App API uses Bearer Auth** (a distinct App API token). Using the wrong credential
type against the wrong host returns a `401`, which can look like a generic
misconfiguration rather than "you used the Track key against the App API." Keeping
these as separate secret fields (rather than one shared credential) avoids the most
common version of this mistake, but it's still worth knowing the failure mode if they
ever get transposed during a deploy or a manual curl test.

## 2. Timestamp units — milliseconds silently misinterpreted as seconds

**This is the highest-value pitfall found in this research pass.** Customer.io's Track
API accepts a millisecond-precision timestamp **without erroring**, but interprets the
number as **Unix seconds** — producing a date thousands of years in the future. The
practical consequence: any segment or automation condition doing a date comparison
against that attribute **silently breaks** (never matches, or matches everything,
depending on the comparison direction) — no error, no warning, just wrong behavior.

**Direct implication for any integration using the Track API**: ensure every
timestamp sent to Customer.io is in **Unix seconds**, not JavaScript's native
millisecond epoch (`Date.now()`) or any other millisecond-precision source. This is
exactly the kind of silent-corruption failure mode — a "200-equivalent" (no error)
masking wrong data — that's expensive to catch after the fact. **Add an explicit
unit-conversion step and a unit test asserting the timestamp is in the expected
second-precision range** rather than trusting call-site discipline alone.

## 3. Data quality — incomplete/inconsistent event payloads break automations quietly

General advice, but directly relevant: an event missing an expected property (e.g. a
malformed enum value) doesn't error at send time — it just produces an automation
branch that silently doesn't fire, or fires down the wrong path. **Generalize the
lesson**: any campaign branch condition keyed on an event or attribute value needs its
upstream emitter covered by an automated test asserting the payload shape — don't add
a new branch condition without extending test coverage to the new field.

## 4. Regional data-out considerations for connected downstream tools

Worth flagging for any project with a data-out integration to a third-party analytics
tool: some downstream integrations have their own independent region setting,
separate from Customer.io's own account region — check both ends of any data-out
connection, don't assume Customer.io's region setting propagates region-correctness
to a connected tool automatically.

## Sources

- [Troubleshooting tips for integrations](https://docs.customer.io/integrations/getting-started/troubleshooting/)
- [Understanding integrations in Customer.io](https://docs.customer.io/integrations/getting-started/how-it-works/)
- [Customer.io data out](https://docs.customer.io/integrations/data-out/connections/customerio/)

Note: several search results pointed at third-party "Agent Skill" packages
(`tonsofskills.com`, `skills.lc`, `atskills.one` — apparently pre-packaged prompt
bundles for coding agents, not Customer.io's own material) that claim to document
"known pitfalls" and "common errors." These were **not** fetched or treated as a
source — they're unverified third-party content of unknown provenance, and this
corpus only records claims traceable to Customer.io's own docs or identifiable
practitioner write-ups with a checkable source.
