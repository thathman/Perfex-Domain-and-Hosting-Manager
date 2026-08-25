# Feature Spec — Renewal, Pricing & Billing Engine

## Status
Committed core capability. Required for MVP.

## Objective
Implement the agreed renewal lifecycle so Domain & Hosting Manager can warn early, open self-service renewal, generate exactly one Perfex invoice, reconcile payment, and track actual provider fulfilment without losing history.

## Default Timeline
For expiry date `E`:
- `E - 180 days`: staff + eligible client renewal notice.
- `E - 60 days`: staff/client notice + open client renewal window.
- `E - 30 days`: create/send Perfex invoice if none exists.
- optional later unpaid steps such as 14/7/1 days.
- `E`: unresolved service/cycle becomes expired/at-risk per policy.

Values are seeded defaults, not hard-coded constants; global policy and service override can change them.

## Early Renewal
If client clicks `Renew Now` at 60–31 days: authorize contact/customer/service; verify window and published price; ensure cycle; atomically claim invoice action; create one Perfex invoice; persist reference; route to invoice/payment. At 30 days, scheduler detects existing invoice/action and skips.

## Pricing Gate
Renewal cannot expose a payable amount or generate an invoice until price is `published`. When unresolved, client sees a friendly pricing-under-review state, staff gets review action, scheduler records blocked/skipped state and creates nothing.

## Pricing Policies
Fixed, manual review, percentage increase, fixed increase, cost-plus percentage, cost-plus fixed and one-off next-cycle override. Guardrails: minimum client price, minimum margin amount/percent, optional Approval Workflow later. Money arithmetic is decimal-safe.

## State Machine
Typical: `upcoming → window_open → invoiced → awaiting_payment → paid → fulfilment_required → processing → renewed`. Alternatives: declined, cancelled, expired, failed. Payment never transitions directly to renewed.

## Invoice Invariants
One primary renewal invoice reference per service cycle; shared invoice allowed for renewal groups but distinct line refs per cycle; client/cron/staff/API all call the same `ensureInvoice`; amount comes from immutable cycle snapshot; ambiguous retries reconcile idempotently before any new create.

## Renewal Groups
Compatible when customer, currency and tax/due-date/payment configuration align and cycles are eligible together. Fulfilment remains per cycle.

## Manual Fulfilment
MVP staff queue shows paid/fulfilment required; authorized staff marks processing/renewed; new expiry required; optional safe provider ref; actor/time audited; next cycle derived. Future provider adapters must use the same completion contract.

## Failure Cases
Email failure does not alter price/invoice state. Invoice failure leaves cycle uninvoiced/retryable with staff visibility. Cancelled/deleted invoice triggers billing review rather than blind recreation. Payment after expiry can still require fulfilment. Provider failure exposes safe staff status and client-friendly state, never raw sensitive error.

## Audit / Events
Audit price preview/publish/change, cycle create, policy action claim/result, Renew Now, invoice create/reuse, payment reconciliation, overrides, fulfilment/new expiry.

Events include price review/published, renewal cycle/window/invoice/payment/fulfilment/completed/failed events under `domain_hosting.*`.

## UI
Staff: Pricing, Pricing Reviews, Renewals, cycle timeline, retries, Mark Processing/Renewed, invoice link, cost/margin only with permission.

Client: renewal date/days, published price, Renew Now / View or Pay / Paid — Processing / Renewed and history.

## Out of Scope for MVP
Automatic registrar renewal, provider-specific payment, contract NLP, advanced churn analytics and support-chat workflow.
