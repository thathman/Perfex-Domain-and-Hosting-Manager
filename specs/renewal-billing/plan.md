# Renewal, Pricing & Billing — Implementation Plan

1. **Schema** — pricing/history, renewal policies/steps/cycles/action ledger/billing groups with constraints/indexes.
2. **Money/Pricing engine** — decimal-safe Money utilities and every committed policy.
3. **Price history** — preview → review_required → publish; immutable history/snapshot.
4. **Renewal cycle service** — deterministic term identity/state machine/date derivation.
5. **Policy engine** — seed 180/60/30, inheritance and service overrides.
6. **Idempotency ledger** — claim before side effect, store result/retry state.
7. **Scheduler** — indexed due-work selection and step handlers.
8. **Billing adapter** — native Perfex invoice adapter + `ensureInvoice`.
9. **Payment reconciliation** — Perfex hooks + repair job.
10. **Notifications** — native templates/merge fields and milestone handlers.
11. **Staff UX** — Pricing Review and Renewal queues/timelines.
12. **Client UX** — eligibility, Renew Now and status/history.
13. **Concurrency/failure tests** — cron/client races, duplicate actions, ambiguous invoice retry.
14. **Staging E2E** — standard renewal, early renewal, pricing-review block and grouped invoice.
