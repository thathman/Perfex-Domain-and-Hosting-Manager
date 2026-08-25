# Renewal, Pricing & Billing — Contracts

## PricingService
`preview(serviceId, input, actor)` returns proposed price, margin, warnings and review requirement without writing. `publish(serviceId, renewalCycleId, proposal, actor)` validates permissions/guardrails, appends price history, snapshots cycle and emits event.

## RenewalService
`ensureCycle(serviceId, actor)` returns existing/new cycle via deterministic term key. `eligibility(serviceId, actor)` returns eligible flag, mode (`renew_now`, `request_renewal`, `unavailable`), window dates, published price, invoice state and reason code. `complete(cycleId, newExpiryDate, providerReference, actor)` records actual fulfilment and emits completed event.

## RenewalActionLedger
`claim(cycleId, actionType, idempotencyKey, fingerprint, actor)` atomically returns a new claim, an existing successful result or conflict. `succeed`, `failRetryable` and `skip` update that ledger row.

## BillingOrchestrator
`ensureInvoice(cycleIds, actor, idempotencyKey)` returns one native Perfex invoice ID plus `created` flag and distinct line references per cycle. It reconciles ambiguous prior outcomes before retrying.

## Notification Handler
`executePolicyStep(cycleId, stepId, actor/system)` uses native Perfex templates and the action ledger.

## Events
Publish only after committed state, with event/correlation/customer/resource IDs and no secrets.
