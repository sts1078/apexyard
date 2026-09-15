# App API — transactional email send

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/journeys/send/transactional/email ·
https://docs.customer.io/messaging/send/transactional/api-examples.md ·
https://docs.customer.io/integrations/api/app/tag/send-messages/sendEmail

## The endpoint

`POST https://api-eu.customer.io/v1/send/email` (App API, `Authorization: Bearer
<APP_API_TOKEN>`) — or the non-EU host if the account isn't EU-region.

## Raw-body send (no vendor template) — confirmed supported

```json
{
  "to": "sarah@example.io",
  "from": "win@customer.io",
  "subject": "Reset your password",
  "body": "<html>...</html>",
  "identifiers": { "id": "d34sh" }
}
```

`subject`, `body` (HTML), `body_plain`, and `from` can all be supplied directly in the
request — **no `transactional_message_id` is required**. This confirms a load-bearing
capability for any design where an application renders its own template (from its own
database, its own render pipeline) and hands Customer.io only the finished bytes,
never a second copy of the template. Customer.io never holds a copy of the content in
that pattern.

## `transactional_message_id` + `auto_create` — metrics label only

```json
{
  "transactional_message_id": "order_confirmation",
  "auto_create": true,
  "to": "sarah@example.io",
  "identifiers": { "id": "sf3sd" },
  "from": "win@customer.io",
  "subject": "Your order is confirmed",
  "body": "..."
}
```

When `transactional_message_id` is passed **alongside** `subject`/`body`, those
request-level fields **override** whatever the template record holds — confirmed by
the docs' own wording ("you may override template values like subject, body, and
sender directly through the API request"). `auto_create: true` creates the metrics
record if the ID doesn't exist yet, and **fails with a 400 if the name is already
associated with a different channel**. This is the pattern for an app-owned-content
design: pass a template key as `transactional_message_id` purely so Customer.io's
reporting UI can attribute deliveries per key, with `auto_create: true` so the first
send creates the label — never as a vendor-held template.

## `send_to_unsubscribed` — the transactional bypass

- Some Customer.io surfaces describe the default as `true`; others describe
  transactional as bypassing suppression **by default** with a **per-message UI
  toggle**, which is a different default-location claim than "default true on every
  API call." **Don't rely on an assumed default for anything with a compliance
  consequence** — set it explicitly on every call where the bypass is actually
  intended, and verify the platform's actual default behavior live against the target
  workspace before depending on it.
- `tracked` (boolean) — controls open/click tracking pixels on the transactional send.
  A design that treats certain message classes as privacy-sensitive (health,
  financial, or otherwise) may want `tracked: false` on those specifically.

## Other fields relevant to a raw-body design

| Field | Purpose |
|---|---|
| `identifiers` | `{ "id": "<person_id>" }` — prefer a stable internal id over email as the identifier, so an email change doesn't fork the profile |
| `disable_css_preprocessing` | Skip Customer.io's CSS inlining — useful when the mail is already fully rendered HTML and the sender doesn't want the vendor reprocessing already-finalized bytes |
| `send_at` | Unix timestamp for scheduled delivery |
| `queue_draft` | Queues as draft instead of sending — useful for testing a message without actually sending |
| `cc` / `bcc` / `reply_to` | Standard email headers, supported |

## What the docs do not resolve (needs a live test)

Several claims need proving against the real workspace, not just the docs: that the
bypass actually delivers past an unsubscribe, that a campaign-originated send to the
same profile is correctly suppressed, and that transactional sends genuinely omit a
`List-Unsubscribe` header while campaign sends carry one. See
`docs/06-subscriptions-topics-unsubscribe.md`.
