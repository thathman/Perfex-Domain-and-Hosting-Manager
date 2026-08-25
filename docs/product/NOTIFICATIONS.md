# Perfex Domain & Hosting Manager — Notification Matrix

## 1. Principles
- Email content uses native Perfex email templates.
- In-app notifications use native Perfex notification infrastructure.
- Client notifications never expose provider cost, margin, internal notes or secrets.
- Staff/client templates are separate where information differs.
- Renewal actions are idempotent; repeated cron runs do not resend a successful milestone.
- Chatwoot/mobile/push are optional downstream channels, not replacements for Perfex delivery.

## 2. Matrix

| Event / Condition | Staff In-App | Staff Email | Client In-App | Client Email | Event/Webhook | Notes |
|---|---:|---:|---:|---:|---:|---|
| Service created | optional | no | no | no | yes | operational |
| Pricing review required | yes | yes/configurable | no | no | yes | staff-only commercial detail |
| Price published/changed | optional | optional | optional | optional price-change notice | yes | client sees published sell price only |
| 180 days before expiry | optional | yes | optional | yes | yes | agreed first renewal notice |
| 60 days before expiry | optional | yes | yes/configurable | yes | yes | renewal window opens |
| Renewal window opened | configurable | combined with 60d | yes | combined | yes | portal action becomes available |
| Client Renew Now | yes | optional | yes | native invoice email | yes | creates early invoice if needed |
| 30-day auto invoice | configurable | native invoice path | yes | native invoice path | yes | skips if early invoice exists |
| Invoice creation failed | yes | yes | no | no | yes | retry/admin attention |
| Optional unpaid milestone | optional | optional | optional | configurable | yes | e.g. 14/7/1 days |
| Payment received | yes | optional | yes | native receipt/payment behavior | yes | cycle becomes fulfilment required |
| Provider renewal required | yes | configurable | no | no | yes | operations queue |
| Renewal processing | optional | no | yes | optional | yes | client-safe status |
| Renewal completed | yes | optional | yes | configurable | yes | new term/expiry |
| Renewal failed | yes | yes | optional friendly | optional friendly | yes | never raw provider error |
| Service expired unresolved | yes | yes | yes | configurable | yes | severity configurable |
| Integration/provider sync failed | admin | admin | no | no | yes | internal |
| Vault link stale | yes | optional | no | no | yes | no secret value |
| Bulk price change applied | yes/audit | optional | no | optional future notice | yes | approval may be required |

## 3. Required MVP Template Keys
- `domain_hosting_renewal_180_client`
- `domain_hosting_renewal_180_staff`
- `domain_hosting_renewal_60_client`
- `domain_hosting_renewal_60_staff`
- `domain_hosting_pricing_review_staff`
- `domain_hosting_renewal_paid_staff`
- `domain_hosting_renewal_completed_client`
- `domain_hosting_renewal_completed_staff`
- `domain_hosting_automation_failed_staff`

Invoice delivery itself uses Perfex's native invoice template/workflow.

## 4. Merge Fields
Client-safe: `{dhm_service_name}`, `{dhm_service_type}`, `{dhm_domain_name}`, `{dhm_provider_name}`, `{dhm_expiry_date}`, `{dhm_days_until_expiry}`, `{dhm_renewal_window_open_date}`, `{dhm_renewal_price}`, `{dhm_previous_price}`, `{dhm_price_difference}`, `{dhm_price_change_percent}`, `{dhm_billing_cycle}`, `{dhm_invoice_number}`, `{dhm_invoice_link}`, `{dhm_client_portal_link}`, `{dhm_renewal_summary}`.

Staff-only: `{dhm_provider_cost}`, `{dhm_margin_amount}`, `{dhm_margin_percent}`, `{dhm_pricing_review_reason}`, `{dhm_internal_renewal_state}`.

Prohibited: passwords, tokens, API keys, EPP/transfer codes, private keys, Vault/OpenBao secrets and raw provider errors with sensitive payload.

## 5. Recipient Rules
Client recipients are active contacts of the service customer who have notice permission/selection and service visibility; never resolve ownership by email string alone. Staff recipients can include assigned staff, account manager, renewal operations, finance for commercial events and admins for automation/integration failures.

## 6. Milestone Idempotency
Each scheduled notification maps to a renewal-action row keyed by renewal cycle + policy step + action type. Successful action is never automatically resent. Manual resend is a new explicit audited action.

## 7. Failure Behavior
Missing template is a visible configuration failure. Transient mail failure follows Perfex/shared retry behavior. No eligible contact is recorded as skipped with reason. Magic Login absence falls back to normal Perfex portal URL. Chatwoot absence has no effect on native delivery.
