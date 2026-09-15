# Data compliance — GDPR, DPA, Sensitive Data, HIPAA guidance

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/integrations/getting-started/data-compliance.md ·
https://docs.customer.io/accounts/settings/data-centers.md ·
https://docs.customer.io/messaging/channels/sms/registration/hipaa-standards.md ·
customer.io/legal/dpa (not independently re-fetched here — a specific project's own
technical design should cite the DPA directly if this is load-bearing for its
compliance case)

## The vendor's Sensitive Data prohibition

Customer.io's DPA defines **Sensitive Data to include "data concerning health,
including PHI"**, and the customer contractually undertakes not to submit it. This is
the constraint any health-adjacent adopter has to route around: a neutral
notification with a neutral subject (no diagnosis, medication, or clinical detail)
contains no data *concerning health* in the DPA's sense, even though the sender
identity may disclose a general relationship (e.g. "this is a clinic"). **Written
vendor confirmation of this reading is worth getting explicitly** — treat any such
reading as unconfirmed until it's in writing from the vendor, not assumed from a
general principle.

## HIPAA guidance — independent confirmation of the content-free-notification pattern

Customer.io's own HIPAA-compliance documentation states a pattern many
health-adjacent designs converge on independently: **"never include PHI directly in
SMS, push, or email messages... send messages containing links to a secure,
authenticated portal where users can safely access their health information."**
Customer.io positions itself as able to act as a HIPAA **Business Associate** for
customers who need that path. This is a genuinely useful finding — a
content-free-notification-plus-authenticated-deep-link design isn't a one-off
workaround, it's the vendor's own documented best practice for exactly this class of
problem. Worth citing directly when asking a vendor to confirm a Sensitive Data
reading in writing.

Note: citing this HIPAA guidance is not itself a claim that HIPAA applies to a given
project (HIPAA is a US regulatory regime; many projects' compliance frame is GDPR or
a different regional regime) — cite it only as independent corroboration of the
pattern, not as a claim that a Business Associate Agreement is needed or relevant.

## GDPR / regional framing

Both the US and EU data centers are described as GDPR-compliant in Customer.io's own
docs, and Customer.io holds an EU-US Data Privacy Framework (DPF) certification for
cross-border transfers. **This framing does not automatically substitute for a
specific residency requirement** a project's own DPO may have set — see
`docs/01-account-regions-eu.md`. Don't let "GDPR compliant" marketing copy be read as
satisfying "EU residency" without checking what the actual legal requirement is.

## Sub-processor list

Automated fetches of `customer.io/legal/sub-processors` have returned a 404 in past
checks. **Request the sub-processor list and a region-scope statement (covering
backups, logs, and support access) directly from the vendor's privacy/legal contact**
rather than relying on a page fetch.

## Practical implication for the specialist agent

Never draft copy, a template, or a data flow that would put a clinical identifier,
diagnosis, medication name, or decline reason into a Customer.io-bound payload —
whether an event property, a `message_data` field, or template content — for any
project whose design depends on staying outside the vendor's Sensitive Data
definition. If asked to help design a message that would violate this, say so plainly
rather than complying, and point at whatever specific compliance decision record the
managed project has for this (check that project's private knowledge layer).
