# Perfex Domain & Hosting Manager — Build Phases

**Planning baseline:** 2026-08-25  
**Delivery principle:** local-first implementation/testing; no GitHub Actions dependency.

## Phase 0 — Foundation, Security Cleanup & Architecture

### Objective
Convert the imported prototype into a safe foundation without implementing speculative advanced features.

### Features
- Establish application/service layer.
- Preserve module slug `domain_manager`.
- Remove active purchase-code/license verification path.
- Remove username/password fields from active forms/views/controllers.
- Hide legacy secret values from API/UI immediately.
- Establish structured configuration, audit and event contracts.
- Establish updater/release scaffolding.
- Establish local test harness and fixtures.
- Add module-level coding conventions and dependency boundaries.

### Dependencies
Perfex core only. Vault is required only if existing plaintext credentials must be migrated rather than discarded.

### Database
- Migration `102` creates new normalized schema and audit/event-support tables.
- Legacy tables remain readable only by migration/import code.
- No destructive secret removal until controlled migration/discard path succeeds.

### APIs
Internal interfaces only: `ServiceCatalogService`, `PricingService`, `RenewalService`, `BillingService`, `AuditService`, `EventPublisher`.

### UI
- No major new production UX yet.
- Legacy credential fields/views disabled.
- Admin warning if legacy secret data is detected and requires secure migration/discard.

### Integrations
Perfex native hooks, Vault migration adapter contract, updater contract.

### Tests
- Module activation.
- Migration 101→102.
- Legacy data preservation.
- Legacy secret non-exposure.
- Permission baseline.
- Updater archive/checksum/version failure tests.

### Exit Criteria
- No new code path writes plaintext credentials.
- Active UI/API cannot reveal legacy passwords.
- New schema exists and is transaction-safe.
- Local tests pass.
- Architecture interfaces are stable enough for Phase 1.

---

## Phase 1 — Core Service Catalog, Providers & Workspace

### Objective
Replace two flat CRUD lists with the durable service/bundle/provider model and modern admin workflow.

### Features
- Service bundles.
- Domain services.
- Hosting services.
- Normalized providers.
- Customer ownership.
- Project/contract associations.
- many-domain-to-one-hosting relationships.
- guided creation workflow.
- service workspace.
- dashboard/search/filter.
- archive lifecycle.
- contextual customer/project entry points.

### Dependencies
Phase 0.

### Database
Services, Bundles, Domain detail, Hosting detail, Providers, Service relationships/associations, Vault links by reference, service status/audit references.

### APIs
Internal CRUD/query contracts implemented through services; public API skeleton may remain disabled until Phase 4.

### UI
- Dashboard.
- Services.
- Domains.
- Hosting.
- Providers.
- 5-step creation wizard.
- workspace tabs: `Overview | Technical | Pricing | Billing & Renewals | Renewal Schedule | Vault | Relationships | Activity`.

### Integrations
Customers, Contacts, Projects, Contracts, optional Vault links.

### Migrations
- Legacy domain/hosting non-secret data import into normalized model.
- Preserve legacy record IDs in migration mapping metadata for traceability.

### Tests
- service/customer boundary.
- provider normalization.
- hosting↔domain many-to-many.
- contextual prefill.
- workspace permissions.
- migration reconciliation counts.

### Exit Criteria
- Every legacy non-secret domain/hosting record is represented or reported as an import exception.
- New records use only new service model.
- giant legacy create/edit forms are no longer primary paths.
- no cross-customer service relationships are possible.

---

## Phase 2 — Pricing, Renewal Cycles & Renewal Scheduler

### Objective
Implement commercial lifecycle, history and the agreed 180/60/30 renewal schedule.

### Features
- provider cost/client price.
- effective-dated price history.
- pricing policies.
- min price/margin guardrails.
- pricing review/publish gate.
- renewal policies and steps.
- renewal cycles.
- immutable cycle snapshots.
- scheduler.
- action idempotency.
- manual renewal completion.
- paid-but-not-renewed queue.
- optional configurable unpaid reminders.
- renewal groups foundation.

### Dependencies
Phase 1.

### Database
Price history/current pricing, renewal policies/steps, renewal cycles, renewal action executions, billing groups/members, integration/job outcome references where shared runtime is unavailable.

### APIs
Internal quote next renewal, publish price, open window, evaluate scheduler, generate cycle, mark renewal complete, retry failed step.

### UI
Pricing tab, Pricing Reviews queue, Renewals queue, Renewal Schedule editor, renewal cycle timeline/history.

### Integrations
Staff notifications, Perfex cron, native activity/audit.

### Tests
- date boundary/timezone behavior.
- duplicate scheduler execution.
- price policy calculations.
- immutable historical snapshots.
- unresolved pricing block.
- concurrent action locks/idempotency.

### Exit Criteria
- 180/60/30 policy evaluates deterministically.
- no email/invoice side effect occurs twice for the same cycle/step.
- pricing and renewal history are immutable after completion except explicit correction workflow.

---

## Phase 3 — Native Billing, Email Templates & Client Portal

### Objective
Deliver the complete business loop from notice to client renewal to Perfex invoice/payment to manual provider fulfilment.

### Features
- native Perfex email templates and merge fields.
- 180-day staff/client notices.
- 60-day staff/client notices.
- automatic 30-day invoice creation/send.
- client portal service cards.
- contact-level view/renew permissions.
- `Renew Now`.
- early renewal invoice creation.
- invoice/payment status.
- renewal history.
- grouped invoice support for compatible cycles.
- payment reconciliation.
- invoice cancellation/deletion reconciliation.
- renewal-complete notifications.

### Dependencies
Phase 2, Perfex invoices/payments/email templates/client portal.

### Database
Only relationship/reference additions if necessary; invoice/payment remain Perfex-owned.

### APIs
Client-facing internal endpoints/actions used by portal. Public client API may remain disabled until Phase 4.

### UI
Client `Domains & Hosting`, renewal cards, pay/view invoice action, status/timeline, customer profile `Domains & Hosting`, project `Infrastructure`.

### Integrations
Customers/Contacts, Invoices/Payments, Email templates, Notifications, optional Magic Login secure client links via its service.

### Tests
- early renewal at day 60.
- double-click/browser-tab renewal.
- cron vs client race.
- invoice uniqueness.
- payment transition.
- contact authorization.
- client data redaction.
- grouped invoice compatibility.

### Exit Criteria
- full MVP client renewal journey works.
- no duplicate invoice can be created.
- client never sees provider cost/internal/secret data.
- payment never auto-marks provider renewal completed.

---

## Phase 4 — Public API, Events, Webhooks & Ecosystem Contracts

### Objective
Expose safe reusable interfaces for modules, external systems and future MCP/Ara without UI-specific coupling.

### Features
- API v1, scopes, pagination/filtering, idempotency keys, error envelope.
- signed outbound webhooks.
- shared webhook delivery/replay where available.
- event catalog and integration logs.
- adapters for Vault, Handoff, Onboarding, Forms, Property Monitor, Chatwoot and Notifications Inbox.

### Database
Avoid bespoke webhook runtime tables if shared integration runtime exists. If unavailable, a minimal local delivery ledger may sit behind an adapter.

### Tests
Scope enforcement, rate limiting, pagination, idempotency, webhook signature/replay, retry/backoff/dead-letter, event schema compatibility, audit correlation.

### Exit Criteria
- API consumers cannot bypass business rules.
- events publish only after committed state.
- failed integrations are visible/replayable.

---

## Phase 5 — Monitoring, SSL & Provider Adapters

### Objective
Add deeper infrastructure awareness without moving source-of-truth boundaries.

### Features
- SSL service type and live certificate metadata/checks.
- provider adapter interface.
- Cloudflare/registrar/hosting/cPanel/Plesk adapters as prioritized.
- provider price/status synchronization.
- Property Monitor/Uptime Kuma context.
- provider renewal API support only where safe and explicitly enabled.

### Dependencies
Phase 4, OpenBao/shared integration runtime for machine secrets.

### Exit Criteria
- core renewal still works with providers disconnected.
- provider failures never corrupt Perfex business state.

---

## Phase 6 — Advanced Workflow & Ecosystem Automation

### Features
- Approval Workflow for price exceptions/bulk adjustments.
- Forms-based service/renewal requests.
- Onboarding requirements/actions.
- Handoff service/bundle association.
- Chatwoot contextual service/renewal actions.
- Organisation Self-Service controls.
- MCP/Ara tools.
- advanced renewal groups.
- estimates/proposals for upgrades.
- richer bulk price adjustments.
- renewal/churn/margin analytics.

### Exit Criteria
Optional integrations enable/disable independently and create no circular install dependency.

---

## Phase 7 — Hardening, Release & Production Readiness

### Features
Performance tuning, accessibility/responsive pass, translation completeness, security review, migration rehearsal, updater rehearsal, backup/rollback test, release ZIP/checksum, documentation finalization.

### Versioning
The repository currently declares module version `1.0.1` while its numeric constant remains `100`, and contains migration `101`. The architecture rewrite is breaking enough to target a new major product version; final exact release number is chosen after implementation/release-state verification. Published tags/releases are never rewritten.

### Release Workflow
1. Build/test locally.
2. Test on local Perfex staging.
3. Produce deterministic module ZIP.
4. Validate archive root/module slug.
5. Calculate SHA-256.
6. Validate declared version/migration version.
7. Create GitHub Release only after candidate passes.
8. Attach ZIP and checksum.
9. Test updater against published release.
10. Promote only after staging validation.

### Exit Criteria
- zero release blockers.
- local/staging test evidence captured.
- no GitHub Actions required.
- release artifact/checksum validated.
- documented rollback path.
