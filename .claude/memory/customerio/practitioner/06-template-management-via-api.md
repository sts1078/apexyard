# Practitioner know-how — template management via API vs. vendor editor

**Fetched:** 2026-09-15
**Sources:** synthesis of `docs/04-app-api-transactional.md` and
`docs/07-campaigns-automations.md`, plus the general lesson any REST-API-fronted
marketing platform teaches when a team migrates onto or off of it.

## The trade-off, stated generally

Every marketing-automation vendor offers a UI-based template/campaign editor as its
main value proposition (non-developers can edit copy without a deploy). Every vendor
also offers an API to manage the same objects programmatically. The trade-off is
always the same shape:

| | Vendor-UI-owned template | App-owned template (rendered, handed to vendor as raw content) |
|---|---|---|
| Who can edit | Marketing, no deploy needed | Requires a PR (or a direct DB edit, if app-rendered from a database) |
| Single source of truth | The vendor holds it | The app holds it; vendor never has a durable copy |
| Drift risk | Two copies (if the app renders something similar elsewhere) | None — one copy, used for every gateway |
| Portability across gateway (e.g. a fallback ESP) | Bytes differ between vendor render and any fallback | Bytes are identical, since the same render function produces what goes to every gateway |

## A common resolution, by message class

- **Marketing**: vendor-editor-owned, matching the vendor's own recommended pattern
  (see `practitioner/01-transactional-vs-campaign.md`) — a good fit when the class has
  no fallback-gateway requirement and benefits from a design/marketing owner's ability
  to iterate without a deploy.
- **Transactional / clinical / billing**: app-owned (rendered from the app's own
  templates), **never** a vendor template — the right call whenever the class has a
  hard fallback-gateway requirement, since a vendor-rendered template cannot be
  reproduced byte-for-byte on a different gateway during a fallback.

## A general lesson worth carrying forward

Whenever a message needs a **fallback gateway** (any two-gateway design, not just
Customer.io-specific), app-owned rendering is close to a hard requirement — a
vendor-rendered template is definitionally gateway-specific (their editor, their
Liquid dialect, their CSS-preprocessing behavior). Don't reach for "author it in the
vendor's UI" for any message class that might need to fail over to a different
sender.

## Practical implication for the specialist agent

If asked to help build a Customer.io campaign in the vendor's editor, that's squarely
in scope for the marketing class. If asked to add a vendor-side template for a message
class a specific project has decided must stay app-owned (check that project's
private knowledge layer for the rule and its rationale), push back and point at the
fallback-gateway reasoning above — the correct action is usually editing the
application's own template source, not creating a vendor-side template object.
