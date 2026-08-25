# Client Self-Service Renewal — Contracts

## Eligibility
`RenewalService::eligibility(serviceId, ContactActor) -> RenewalEligibilityDto`

Fields: eligible, mode, expiry date, window-open date, published price, currency, invoice ID/status, renewal state and client-safe reason code.

## Renew
`RenewalService::renew(cycleId, RenewCommand, ContactActor) -> RenewalResult`

Command contains only idempotency key and request/correlation ID. Customer, amount, provider cost and all authoritative values are loaded server-side.

Result: cycle ID, invoice ID, created/existing flag, safe payment URL and state.

## Portal Query
`ClientServiceQuery::listForContact(contactId, filters)` derives customer and scopes repository query before retrieval.

## Events
Use common Domain Manager event envelope; client-origin events identify actor type `contact` and carry customer/correlation IDs.

## Client-Safe Errors
- `DH_FORBIDDEN` → not authorized.
- `DH_RENEWAL_WINDOW_CLOSED` → renewal not available yet.
- `DH_PRICE_NOT_PUBLISHED` → renewal price is being prepared.
- `DH_INVOICE_CREATE_FAILED` → invoice could not be prepared; staff notified.
- `DH_CONFLICT` → state changed; refresh/retry safely.
