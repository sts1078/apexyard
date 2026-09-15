# AI features & settings

**Fetched:** 2026-09-15
**Source:** https://docs.customer.io/accounts/security/ai-settings.md ·
https://docs.customer.io/experimental-features · https://docs.customer.io/release-notes ·
https://docs.customer.io/journeys/send/workflows/llm-actions

## The account-level AI toggle

A single **"Customer.io AI"** toggle in Account Settings → Privacy, Data & AI enables
or disables **all** in-app AI features across every workspace in the account
(translation, segment-building suggestions, LLM workflow actions, content-safety
filtering config, etc.). Disabling it does **not** affect the separate MCP server
configuration — MCP and the in-app AI toggle are independent controls.

A project with a region-restricted compliance posture (e.g. EU-only data processing)
should confirm whether AI processing genuinely stays within the account's
data-center region or is a separately-scoped setting — the docs excerpts gathered
don't show a distinct "region-restricted AI processing" toggle separate from the
account's own region, so this is worth verifying live rather than assuming.

## Content-safety filtering (Gemini-backed)

AI features are described as using Gemini models. Admins can configure filtering across
four categories — Harassment, Hate Speech, Sexually Explicit, Dangerous Content — with
levels from "no filtering" to "strict blocking." **No filtering is applied by default**
— if AI features are used at all (translation, segment suggestions), an admin should
actively configure the filtering level rather than assume a safe default.

## What AI features exist

| Feature | What it does | Notes |
|---|---|---|
| AI-generated attribute/event descriptions | Auto-documents the Data Index | Cosmetic; low priority |
| AI-built segments | Natural-language prompt → segment condition suggestion | Could help a marketing/design owner draft a segment faster; **always requires human review before saving** per the docs' own caveat |
| AI translation | Auto-translates message content per locale | Not a fit for message classes an application renders and translates itself via its own pipeline. Marketing content built in the vendor's editor is a more natural candidate, subject to review — the docs explicitly note it "is not currently available for emails created in Design Studio," so confirm compatibility with whatever editor surface is actually in use. |
| LLM workflow actions | Generate text / make decisions inside an automation, using account AI settings + Liquid-accessible data | Can access profile/event attributes but not external sites or media |

## Practical implication for the specialist agent

For any project with a region-restricted or Sensitive-Data-sensitive compliance
posture, **do not recommend or configure any Customer.io AI feature without first
confirming it respects the required data-processing scope** — this is a live item to
verify, not a settled fact from the docs alone. When in doubt, recommend the AI
toggle stay off for any workflow touching sensitive or identifying content, regardless
of whether the specific feature claims not to see that content — the account-level
Sensitive Data prohibition (see `docs/11-data-compliance-gdpr-dpa.md`) should be read
as covering AI-assisted processing too, absent an explicit vendor carve-out.
