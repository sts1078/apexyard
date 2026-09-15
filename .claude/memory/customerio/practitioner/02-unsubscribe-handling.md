# Practitioner know-how — unsubscribe handling patterns

**Fetched:** 2026-09-15
**Sources:** Customer.io docs (cited inline) and general ESP-integration practice.
A specific managed project's own unsubscribe design (if it has one) is authoritative
for that project — check its private knowledge layer; this file explains the general
pattern and why it tends to hold up, not one project's specific implementation.

## The core practitioner pattern: never trust a vendor's suppression list as the source of truth

The single most important lesson from general ESP practice (and echoed by more than
one project's own post-mortem after trusting a vendor's built-in consent gating too
much): **a vendor's own suppression/unsubscribe state must never be the system of
record for consent.** Reasons that generalize across platforms:

1. **Multiple entry points exist and will diverge if not centralized.** One-click
   headers, footer links, spam complaints, in-app settings, and a vendor's own
   preference center are all distinct paths into "this person doesn't want mail." If
   two of them write to different places, they will eventually disagree, and someone
   gets mailed after unsubscribing (or vice versa).
2. **A vendor-native unsubscribe is usually global, not purpose-scoped.** Confirmed
   for Customer.io specifically (`docs/06-subscriptions-topics-unsubscribe.md`): the
   RFC 8058 one-click header always unsubscribes globally. A design that needs
   per-purpose granularity (marketing off, other classes still on) cannot rely on the
   vendor's own one-click semantics alone — it needs an app-owned ledger plus careful
   header design (only render `List-Unsubscribe` on marketing sends).
3. **Consent withdrawal must be checked at send time, not at list-build time** — the
   race a pre-computed recipient list creates (a withdrawal during an in-flight batch
   still sends if checked only when the list was built). This is a general
   ESP-integration lesson: any system that pre-computes a recipient list ahead of the
   actual send has this race.

## A four-layer containment pattern worth generalizing

1. **Own link, one write path** — every marketing template carries an app-owned
   signed-token unsubscribe link; the vendor's own `{% unsubscribe_url %}` /
   subscription center are never rendered.
2. **Projection filters** — the app pushes a consent attribute at `identify` time;
   every campaign's entry filter reads that attribute (confirmed real mechanism per
   `docs/07-campaigns-automations.md`).
3. **Push withdrawals into the vendor too** — a ledger withdrawal also sets the
   vendor's own `unsubscribed = true` attribute, defence-in-depth for a hand-built
   campaign that might not have the filter wired correctly.
4. **Ingest the vendor's own webhook** — `email_unsubscribed` / `email_spam_reported`
   events get written back through the **same** write function the app's own link
   uses, so there is one write path regardless of entry point.

## A pattern worth generalizing to any future ESP-webhook consumer

**A spam complaint is not merely a stronger unsubscribe — it should trigger
withdrawal of ALL marketing purposes, not just the one the complained-about message
belonged to.** A complaint damages sender reputation platform-wide, so scoping its
effect to one purpose under-reacts to the signal.

## Where this is genuinely uncertain and needs a live check

Whether Customer.io forces `List-Unsubscribe` onto transactional sends regardless of
intent (some ESPs do, for deliverability-policy reasons unrelated to the sender's
preference) is **not settled by documentation** — it's the kind of claim that needs a
live test against the real workspace. If it turns out Customer.io forces the header,
a reasonable fallback is recording it as a marketing-only withdrawal (safe direction),
not treating it as a design failure.
