# Practitioner know-how — EU/GDPR considerations for a health-adjacent ESP integration

**Fetched:** 2026-09-15
**Sources:** synthesized from general ESP-vendor-evaluation practice, Customer.io's
documented account-region and HIPAA guidance, and the recurring pattern seen across
more than one health-adjacent adopter's vendor-selection history. This file records
the *general pattern*; a specific managed project's own DPO rulings and decision
records are authoritative for that project — check its private knowledge layer.

## A pattern worth generalizing from real vendor-evaluation history

Adopters that cycle through more than one ESP before landing on a final choice tend to
converge on the same underlying insight:

1. **"GDPR compliant" (vendor marketing claim) ≠ "meets our residency requirement"
   (a specific legal ruling).** Every hosted ESP will claim GDPR compliance. The
   question that actually matters is narrower and specific to a DPO's ruling: *where
   is the data physically stored, and does that satisfy the prerequisite the DPO set?*
   Some rulings require EU data residency specifically, not "any GDPR-adequate
   transfer mechanism." Never assume "the vendor says GDPR compliant" settles a
   residency question without checking what the actual legal requirement is for the
   specific project in question.

2. **A vendor's acceptable-use / Sensitive-Data clause can eliminate an otherwise-good
   platform choice outright**, independent of the residency question. Customer.io's
   own contract (like most hosted ESPs) prohibits health data in message content. This
   is not negotiable by engineering effort — the fix has to be architectural (don't put
   health data in the payload at all).

3. **Content-free notification + authenticated deep-link is a generalizable pattern**
   for exactly this class of constraint, and — independently confirmed — is
   Customer.io's own documented HIPAA-compliance guidance (see
   `docs/11-data-compliance-gdpr-dpa.md`). When a vendor's terms forbid the content a
   message would otherwise carry, moving the content behind authentication and sending
   only a neutral pointer is a well-trodden path, not an idiosyncratic workaround.

## Consent must be app-owned, never vendor-owned, for any GDPR-scoped marketing

Regardless of which ESP is in play, consent state (opt-in, opt-out, double opt-in
proof) belongs in the app's own ledger, with the vendor receiving only a read
projection (a consent attribute). A vendor whose consent gating rests on segment
membership is only as trustworthy as that vendor's sync/rebuild cadence — a vendor's
segment or list is a **cache**, not a **source of truth**. Never design a
consent-check that reads a vendor's state directly without an app-owned ledger
backing it.

## Evidence discipline — verify live, don't trust documentation alone, for claims with legal consequence

A disciplined design separates "settled by documentation" from "must be verified
against the live workspace" for every claim with legal or safety consequence — the
bypass flag, the ordering guarantee, the header behavior. This is the right
discipline for any integration where a wrong assumption has a compliance
consequence, not just a bug consequence. This agent should maintain the same
separation when advising: cite a documentation-sourced claim as "the docs say X" and a
live-verified claim as "confirmed against the workspace on \<date\>" — never blur the
two, especially for anything touching consent, suppression, or a vendor's
Sensitive-Data boundary.
