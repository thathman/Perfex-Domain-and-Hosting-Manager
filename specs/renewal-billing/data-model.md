# Renewal, Pricing & Billing — Data Model

Canonical definitions are in `docs/product/DATA_MODEL.md`.

This feature uses:
- `domain_manager_service_pricing`
- `domain_manager_price_history`
- `domain_manager_renewal_policies`
- `domain_manager_renewal_policy_steps`
- `domain_manager_renewal_cycles`
- `domain_manager_renewal_actions`
- `domain_manager_billing_groups`
- `domain_manager_billing_group_members`

## Required Unique Constraints
1. `renewal_cycles(service_id, term_key)`
2. `renewal_actions(idempotency_key)`
3. deterministic policy-action uniqueness for cycle/step/action
4. stable invoice-line-reference uniqueness.

## Snapshot Rule
At price publication/invoice eligibility, cycle captures currency, provider cost, client price, previous expiry, target dates and pricing-policy provenance. Later service price edits do not mutate the cycle.

## Retention Rule
Renewal cycles and price history are business/financial history and are retained/archived, not deleted during normal service lifecycle.
