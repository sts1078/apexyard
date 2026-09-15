# Sending domains, DKIM/DMARC, link tracking, deliverability

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/messaging/channels/email/deliverability/best-practices.md ·
https://docs.customer.io/release-notes · https://docs.customer.io/journeys/channels/links/universal-links

## Domain authentication — required, per-workspace

MX, DKIM, and SPF records must be configured **per workspace** in workspace settings
to authorize Customer.io to send on behalf of a domain — this is described as "the
most critical step" for inbox providers to recognize the domain as legitimate. Any
multi-workspace setup (e.g. staging/prod) means duplicated setup, not a one-time
account-level task — don't assume a second workspace inherits the first one's domain
authentication.

## DMARC — now actively checked

Customer.io validates DMARC policy status during sending-domain verification, in
response to updated Gmail/Yahoo bulk-sender requirements (effective across the
industry since early 2024). Check DMARC policy status in Workspace Settings and follow
any update recommendations shown there — a misconfigured or absent DMARC record is
now a deliverability blocker, not just a best-practice suggestion.

## Link tracking

Link tracking setup was simplified to remove the need to manage your own SSL
certificates or a reverse proxy (NGINX/Cloudflare) — Customer.io now handles the
tracking infrastructure, configured similarly to setting up a sending domain. For a
custom tracking domain (`email.yourdomain.com`-style), a client-side snippet can
extract the `link_id` and independently POST to record a click — relevant only if a
project needs non-native link tracking (e.g. tracking clicks from a channel Customer.io
doesn't natively instrument); a design that deliberately disables tracking on certain
sensitive message classes wouldn't need this.

## Manual metric reporting for non-native sends

`POST https://track.customer.io/api/v1/metrics` lets you report delivery metrics
(`bounced`, `clicked`, `converted`, `deferred`, `delivered`, `dropped`, `opened`,
`spammed`) for messages sent through a channel that isn't native to Customer.io —
keyed by the `CIO-Delivery-ID` header from the original notification. Not directly
relevant to a design where a non-Customer.io fallback gateway (e.g. a direct SES
integration) has its own independent delivery-tracking chain, but worth knowing it
exists if a future gap needs bridging.

## RFC 8058 one-click compliance

Gmail and Yahoo have required RFC 8058 one-click unsubscribe (`List-Unsubscribe` +
`List-Unsubscribe-Post`) for bulk senders since early 2024. A design with its own
signed-token unsubscribe endpoint (rather than the vendor's) should implement this
pair pointing at its own endpoint — see `docs/06-subscriptions-topics-unsubscribe.md`.
Sending without it, or without a valid DMARC record, risks deliverability degradation
independent of any consent/compliance question — a second, orthogonal reason to get
both right.
