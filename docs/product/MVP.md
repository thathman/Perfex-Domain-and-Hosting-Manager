# Perfex Domain & Hosting Manager — MVP

**Status:** Committed implementation scope  
**Target:** First production-capable release on the new architecture

## 1. MVP Definition

The MVP is not a CRUD rewrite. It must establish the durable service, pricing, renewal, billing, client-self-service, security, API/event and updater architecture needed for the full product.

A release is not MVP-ready merely because staff can store domains/hosting records.

## 2. Included Features

### Foundation
- Keep module slug `domain_manager`.
- Remove legacy purchase-code gating from the active product path.
- Introduce service-layer architecture separate from controllers/views.
- Contiguous migrations beginning after the current `101_version_101.php`.
- Structured audit.
- Event dispatcher contract.
- Safe updater/release scaffolding based on the Magic Login hardened direction.
- Local-only test execution; no GitHub Actions dependency.

### Service model
- Service bundles.
- Domain service type.
- Hosting service type.
- Normalized providers.
- Customer ownership.
- Contact visibility/notification targeting.
- Project relationships.
- Contract relationship.
- Multiple domains per hosting account.
- Status/archive lifecycle.
- Search/filter/admin dashboard.

### Admin UX
- Guided 5-step create workflow:
  1. type;
  2. client/relationships;
  3. service details;
  4. pricing/renewal;
  5. review.
- Post-create service workspace.
- Customer `Domains & Hosting` tab.
- Project `Infrastructure` tab.
- Renewal queue.
- Pricing-review queue.
- Provider management.
- Activity/audit view for authorized staff.

### Pricing
- Provider cost and client price.
- Currency and billing cycle.
- Effective-dated price history.
- Price policies:
  - fixed;
  - manual review;
  - percentage increase;
  - fixed increase;
  - cost-plus percentage;
  - cost-plus fixed amount;
  - one-off next-renewal override.
- Minimum price / minimum margin guardrails.
- Published-price gate.
- Price-change preview.
- Provider-cost change can require review.
- Immutable renewal-cycle price snapshot.

### Renewal policy engine
Seeded default policy:
- 180 days: email staff + client.
- 60 days: email staff + client + open client renewal.
- 30 days: create/send Perfex invoice if none exists.

Also:
- policy inheritance;
- per-service override;
- idempotent action execution;
- configurable optional post-30-day reminder milestones;
- pricing-ready prerequisite;
- renewal cycle history;
- manual provider-renewal completion;
- expiry/at-risk state;
- retryable failures.

### Client self-service
- Native Perfex portal page/cards.
- Contact-level view permission.
- Contact-level renewal permission.
- Default renewal window starts 60 days before expiry.
- `Renew Now` before the scheduled invoice date creates the invoice immediately.
- Existing unpaid invoice results in `View / Pay Renewal Invoice`.
- Paid invoice shows processing state until actual renewal completion.
- Renewal history.
- No provider costs/internal notes/secrets.

### Billing
- Native Perfex invoice creation.
- Invoice line carries stable renewal/service reference metadata.
- Idempotent invoice creation.
- Payment reconciliation.
- Renewal groups capable of producing one invoice with multiple renewal line items where customer/currency/tax rules are compatible.
- Invoice cancellation/deletion reconciliation behavior.

### Notifications
Native Perfex email templates and in-app notifications for:
- 180-day client/staff notice;
- 60-day client/staff notice;
- invoice created/sent;
- pricing review required;
- paid / provider action required;
- renewal complete;
- automation failure.

### Vault / secrets
- No password or API-secret fields in new schema/forms/APIs.
- Vault reference linking for human/client credentials.
- OpenBao reference model reserved for machine/provider integrations.
- Legacy plaintext credentials hidden immediately from new UI/API and handled by controlled upgrade process.

### API/events
- Internal service contracts for services, pricing, renewal and billing.
- Public API v1 read/write/action surface for permitted use cases.
- Scopes and authorization.
- Pagination/filtering.
- Idempotency keys for mutation actions that can duplicate side effects.
- Stable error contract.
- Core webhooks/events.

### Audit/observability
- Actor/action/resource/previous/result/timestamp/correlation data.
- Scheduler and integration execution logs.
- Failure visibility and replay entry points.
- No secret values in logs.

## 3. Deferred from MVP

These are deferred, not rejected:

- Automated SSL certificate discovery/checking.
- SSL as fully managed service type.
- DNS/server/business-email service types.
- Registrar/provider API adapters.
- Automatic provider-side renewals.
- cPanel/WHM/Plesk automation.
- Cloudflare/Hostinger-specific adapters.
- Uptime Kuma live health widgets.
- Property Monitor operational health aggregation.
- Approval Workflow integration for price exceptions/bulk changes.
- Forms-driven public service intake.
- Advanced contract policy enforcement.
- Estimates/proposals generated from service upgrades.
- Subscription-first billing models.
- Full bulk price adjustment execution; MVP may include review/preview and safe limited bulk actions.
- AI/MCP action execution beyond exposing stable internal contracts.
- Chatwoot conversational renewal workflows.
- Advanced analytics/churn forecasting.
- Cross-customer shared hosting constructs.
- Automated domain transfer workflows.

## 4. MVP Dependencies

### Hard dependencies
- Supported Perfex CRM version.
- Native Perfex Customers, Contacts, Projects, Contracts, Invoices, Payments, Email Templates, Notifications, Cron and permissions.
- PHP/runtime versions supported by the deployed Perfex instance.
- Database supported by Perfex.

### Required when legacy secrets exist during upgrade
One of:
- Perfex Vault available for controlled migration; or
- explicit authorized discard of legacy credential values after backup.

The new product must not continue exposing or writing plaintext credentials.

### Optional integrations
- Perfex Vault for new credential references.
- Perfex Handoff.
- Perfex Onboarding.
- Perfex Forms.
- Perfex Property Monitor.
- Uptime Kuma.
- Approval Workflow.
- Notifications Inbox.
- Chatwoot.
- Magic Login.
- OpenBao/provider runtime.
- MCP/Ara.

None may create a circular install dependency.

## 5. MVP Launch Blockers

Release is blocked if any of the following is true:

1. Duplicate invoices can be produced by concurrent cron/client actions.
2. Client can see another customer’s service or renewal.
3. Client can see provider cost/margin/internal notes/Vault secret data.
4. Pricing-unresolved cycles can produce invoices.
5. Payment marks a renewal completed before provider renewal.
6. Legacy plaintext passwords remain visible in the active UI/API.
7. New schema stores plaintext secrets.
8. 180/60/30 actions are not replay-safe.
9. Email bodies are hard-coded instead of using Perfex templates.
10. Migration chain is discontinuous.
11. Upgrade can corrupt existing non-secret data.
12. Updater can overwrite files without checksum/archive/version validation and backup.
13. No audit entry exists for price changes, invoice generation, renewal completion and permission-sensitive actions.
14. API actions bypass the same authorization/business rules as UI actions.
15. External/webhook processing lacks signature/idempotency controls where applicable.
16. Client portal is an exposed admin table instead of a purpose-built experience.
17. Automated tests do not cover concurrency/idempotency and tenant boundaries.

## 6. MVP Acceptance Criteria

### Service creation
- Domain and Hosting can be created through the guided workflow.
- Customer is mandatory.
- Hosting can link to multiple domain services of the same customer.
- Contextual customer/project entry points prefill relevant values.

### Pricing
- Authorized user can record provider cost and client price.
- Price changes produce history rows.
- Next-renewal price can be computed from supported policies.
- Price can be put into review and published.
- Provider cost is absent from all client responses.

### Renewal
- System creates one renewal cycle for an upcoming term.
- 180-day notice executes once.
- 60-day notice executes once and opens renewal.
- Client can renew at 60 days.
- Early renewal produces exactly one invoice.
- 30-day scheduler detects the invoice and skips creation.
- If client did not renew early, 30-day scheduler creates/sends one invoice.
- Failed action can be retried without duplicates.

### Payment/fulfilment
- Perfex payment causes cycle to become paid/awaiting fulfilment.
- Staff sees paid-but-not-renewed queue.
- Authorized staff records new expiry and marks renewal complete.
- Next cycle can be derived without modifying prior history.

### Security
- No active password fields.
- Vault links expose reference metadata only.
- Customer and contact permission tests pass.
- CSRF/XSS/SQL injection regression tests pass.
- Sensitive logs contain no credentials.

### Release
- Local test suite passes.
- Staging upgrade from current repository baseline passes.
- Release ZIP structure validated.
- SHA-256 published and validated.
- Semantic version check passes.
- Rollback backup is created before upgrade.
