# Feature Spec — Client Self-Service Renewal

## Status
Committed core capability. Required for MVP.

## Objective
Allow authorized Perfex client contacts to understand and act on eligible domain/hosting renewals from the native Perfex client portal without exposing internal commercial, credential or operational data.

## Default Eligibility
Service becomes self-renewable 60 days before current expiry when service is renewable, self-renew is enabled, authenticated contact belongs to the customer, contact has renewal permission, cycle exists, price is published and no blocking approval/state exists.

## Portal States
- **Before window:** show service/expiry, no Renew Now; may show open date.
- **Window open/no invoice:** expiry, days, billing cycle, published price, Renew Now.
- **Invoice unpaid:** invoice status + View / Pay Renewal Invoice.
- **Paid/fulfilment pending:** Renewal Paid — Processing.
- **Processing:** safe progress only.
- **Renewed:** new expiry/term and renewal history.
- **Pricing unresolved:** Renewal available soon; no amount/action; staff pricing-review alert.
- **Request-only mode:** Request Renewal if approval is required; no invoice until approved.

## Renew Now Transaction
Authenticate contact; derive customer from session; customer-scope cycle load; check service visibility and renew permission; validate window/service/published price; call BillingOrchestrator with idempotency key; return existing/created invoice; route to native Perfex invoice/payment.

Browser-supplied amount, provider cost or customer ID is never authoritative.

## Contact Permissions
Separate service view, renewal-history view, renew and receive-notice permissions. Organisation Self-Service may later map org roles to them.

## Data Redaction
Never expose provider cost/margin, pricing-policy internals, internal notes/errors, Vault/OpenBao IDs/secrets, audit/correlation details or provider machine-account details not explicitly client-safe.

## Mobile UX
Renewal card/date/price/action/invoice/status/history must be usable on mobile; avoid admin-style dense tables.

## Notifications
180-day notice may link to service but does not open renewal. 60-day notice links to open renewal. Invoice email uses native Perfex invoice flow. Completion may use module template. Magic Login can optionally supply secure links through its own contract; otherwise normal portal URL.

## Failure Behavior
Duplicate submits and cron/client races yield one invoice. Invoice-service outage gives friendly client error and staff/retry visibility. Permission revoked after page load still denies action. Price change after invoice creation does not mutate cycle/invoice snapshot absent explicit correction.

## Events
`domain_hosting.renewal.window_opened`, invoice requested/created, payment received and renewal completed.

## Out of Scope
Provider-side renewal from browser, credential display, client price-negotiation chat and custom payment rails outside Perfex invoice/payment architecture.
