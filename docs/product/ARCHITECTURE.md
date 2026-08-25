# Perfex Domain & Hosting Manager — Architecture

## 1. Architectural Position

Perfex CRM is the primary business system and source of truth. Domain & Hosting Manager is a self-contained Perfex module that owns the managed-infrastructure service lifecycle and references other Perfex/module-owned objects through stable IDs/contracts.

The module must not patch Perfex core.

## 2. Dependency Direction

```text
UI controllers / client controllers / API controllers
                |
                v
        Application services
                |
        +-------+--------+------------------+
        |                |                  |
        v                v                  v
 Service Catalog     Pricing Service    Renewal Service
        |                |                  |
        +---------> Billing Orchestrator <--+
        |
        v
 Repositories / Models
        |
        v
      MySQL

Application services ---> Port/Adapter interfaces ---> Perfex native services
                                          |---------> Vault
                                          |---------> OpenBao runtime
                                          |---------> shared events/webhooks
                                          |---------> provider adapters
```

Cross-module code should depend on interfaces/events, not reach into another module's database tables.

## 3. Components

### 3.1 Bootstrap / module registration
Responsibilities:
- register module metadata;
- register admin/client hooks;
- register permissions;
- register email merge fields;
- register cron callback;
- register API routes through shared API mechanism;
- register activation/migration/updater hooks;
- register optional integration adapters when dependencies are available.

No business logic belongs in bootstrap functions.

### 3.2 Controllers

#### Admin controllers
Thin request/response adapters for:
- dashboard;
- services/bundles;
- providers;
- pricing reviews;
- renewals;
- settings;
- audit/integration diagnostics.

#### Client controllers
Purpose-built portal endpoints:
- customer-scoped service list/detail;
- renewal eligibility;
- renew action;
- renewal history.

#### API controllers
Versioned endpoints that invoke the exact same application services as UI controllers.

Controllers must:
- authenticate;
- authorize;
- validate request DTOs;
- call application service;
- map service result/errors to response.

Controllers must not:
- create invoices directly;
- calculate pricing independently;
- directly mutate cross-module tables;
- read Vault/OpenBao secret values.

### 3.3 Application Services

#### `ServiceCatalogService`
Owns:
- service/bundle creation/update/archive;
- relationships;
- provider assignment;
- type-specific validation;
- customer-boundary enforcement.

#### `PricingService`
Owns:
- current price/cost;
- price policy evaluation;
- effective-dated price history;
- review/publish;
- margin guardrails;
- bulk preview;
- renewal-price snapshot.

#### `RenewalService`
Owns:
- renewal cycle creation;
- renewal-policy inheritance;
- eligibility;
- milestone evaluation;
- state transitions;
- manual completion;
- cancellation/failure transitions;
- next-cycle derivation.

#### `RenewalScheduler`
Selects due work and delegates to `RenewalService`. It never duplicates business rules.

#### `BillingOrchestrator`
Owns:
- renewal invoice request;
- grouped invoice composition;
- idempotency;
- Perfex invoice references;
- payment reconciliation;
- invoice cancellation/deletion reconciliation.

It never becomes an invoice source of truth.

#### `NotificationService`
Maps domain events/milestones to native Perfex notification and email-template sends.

#### `AuditService`
Writes structured audit records and native human-readable activity log entries.

#### `EventPublisher`
Publishes committed domain events into shared event/webhook runtime.

#### `LegacyImportService`
Only used for migration/import of current `domain_manager` / `hosting_details` data.

#### `ProviderGateway`
Interface for optional providers. Manual gateway is always available.

### 3.4 Repositories / Models

Use focused repositories/models:
- ServiceRepository.
- BundleRepository.
- ProviderRepository.
- DomainDetailsRepository.
- HostingDetailsRepository.
- ServiceRelationshipRepository.
- PricingRepository.
- RenewalPolicyRepository.
- RenewalCycleRepository.
- RenewalActionRepository.
- BillingGroupRepository.
- VaultLinkRepository.
- AuditRepository.

Avoid a single god-model.

## 4. Transaction Boundaries

### Create service
One transaction:
- service;
- type details;
- primary relationships;
- pricing baseline;
- renewal policy association;
- audit staging.

Emit event only after commit.

### Publish renewal price
One transaction:
- new price-history row;
- renewal-cycle snapshot/update;
- review state;
- audit.

### Generate invoice
Use a database-side idempotency lock/unique key before invoking Perfex invoice creation. Store a durable attempt/action row. When Perfex invoice is created:
- persist invoice reference;
- mark action success;
- audit;
- emit event.

Failure leaves an actionable retry state.

### Complete renewal
One transaction:
- validate payment/override policy;
- set actual provider-renewal completion;
- record new expiry;
- finalize cycle;
- derive next service expiry/current state;
- audit.
Then emit `domain_hosting.renewal.completed`.

## 5. Jobs / Cron

### Renewal scheduler
Runs through Perfex cron at a safe cadence following Perfex conventions.

Responsibilities:
- identify services/cycles crossing policy thresholds;
- ensure cycle exists;
- claim step via unique idempotency key;
- execute notification/invoice/window-open actions;
- record result.

Must be safe when repeated.

### Payment reconciliation
Prefer native Perfex hooks/events. A periodic reconciliation job can repair missed hook events.

### Retry worker
Processes retryable failed integration/notification/invoice actions with exponential backoff where applicable.

### Provider sync
Deferred. Uses generic provider adapter and shared integration runtime.

## 6. Events

Event namespace: `domain_hosting.*`

Core:
- `domain_hosting.bundle.created`
- `domain_hosting.bundle.updated`
- `domain_hosting.service.created`
- `domain_hosting.service.updated`
- `domain_hosting.service.archived`
- `domain_hosting.relationship.created`
- `domain_hosting.price.review_required`
- `domain_hosting.price.published`
- `domain_hosting.price.changed`
- `domain_hosting.renewal.cycle_created`
- `domain_hosting.renewal.notice_sent`
- `domain_hosting.renewal.window_opened`
- `domain_hosting.renewal.invoice_requested`
- `domain_hosting.renewal.invoice_created`
- `domain_hosting.renewal.invoice_failed`
- `domain_hosting.renewal.payment_received`
- `domain_hosting.renewal.fulfilment_required`
- `domain_hosting.renewal.completed`
- `domain_hosting.renewal.failed`
- `domain_hosting.service.expired`
- `domain_hosting.integration.failed`

Event envelope:
```json
{
  "event_id": "uuid",
  "event_name": "domain_hosting.renewal.completed",
  "event_version": 1,
  "occurred_at": "2026-08-25T00:00:00Z",
  "correlation_id": "uuid",
  "actor": {"type": "staff", "id": 123},
  "resource": {"type": "renewal_cycle", "id": 456},
  "customer_id": 10,
  "data": {}
}
```

No secret values.

## 7. Integration Ports

### Perfex Customers/Contacts
Read customer/contact identity and authorization context. Domain Manager stores stable IDs only.

### Perfex Projects
Reference project IDs. Project tab calls module services.

### Perfex Contracts
Reference contract IDs and structured pricing-rule metadata owned by this module where needed. Do not parse contract bodies as authoritative policy.

### Perfex Invoices/Payments
BillingOrchestrator calls native invoice services. Store invoice IDs only. Payment hooks update renewal state.

### Perfex Vault
`VaultReferenceGateway`:
- link reference;
- validate existence/permission;
- produce safe deep link.
No secret reveal method in Domain Manager contract.

### OpenBao
`MachineSecretReferenceGateway`:
- provider adapter stores a secret reference/alias only;
- actual token retrieval is handled by common integration broker/runtime, not ordinary module controllers.

### Shared events/webhooks
Publish domain events into shared event delivery. If unavailable, use an adapter that can later be swapped without changing domain services.

## 8. Client Authorization Boundary

Client contact request:
1. authenticate through native Perfex;
2. resolve customer/contact;
3. evaluate Domain Manager contact permissions;
4. scope repository query by `customer_id` before retrieval;
5. enforce service visibility;
6. redact staff-only fields at DTO/serializer layer.

Never fetch unrestricted resource then only hide it in the view.

## 9. Staff Authorization Boundary

Permissions checked in application services, not only menus/controllers.

Sensitive distinctions:
- view commercial data;
- view provider cost/margin;
- publish price;
- generate/reissue renewal invoice;
- complete renewal;
- configure policies/providers/integrations;
- access audit;
- export.

## 10. Secret Boundaries

### Vault
Human/client credentials.

### OpenBao
Machine/service/provider API secrets.

### Domain Manager
Only stable reference IDs, non-secret provider/account labels, safe URLs/metadata and capability flags.

Prohibited:
- passwords;
- API tokens;
- EPP/transfer codes;
- private keys;
- session cookies;
- secret values in logs/merge fields/events.

## 11. Error Handling

Application error codes:
- `DH_INVALID_ARGUMENT`
- `DH_FORBIDDEN`
- `DH_NOT_FOUND`
- `DH_CONFLICT`
- `DH_PRICE_NOT_PUBLISHED`
- `DH_RENEWAL_WINDOW_CLOSED`
- `DH_INVOICE_ALREADY_EXISTS`
- `DH_RENEWAL_ALREADY_COMPLETED`
- `DH_INVOICE_CREATE_FAILED`
- `DH_DEPENDENCY_UNAVAILABLE`
- `DH_INTEGRATION_RETRYABLE`
- `DH_INTEGRATION_TERMINAL`

UI maps these to friendly messages. API returns a stable envelope.

## 12. Integration Failure Behavior

### Perfex invoice failure
- renewal cycle remains un-invoiced;
- action marked failed/retryable;
- staff notification/admin queue entry;
- no duplicate on retry.

### Email failure
- action records failed delivery;
- retry follows Perfex mail/shared-runtime behavior;
- failure is visible;
- invoice state unaffected.

### Vault unavailable
- service operations not requiring Vault continue;
- Vault association/deep-link actions show dependency unavailable;
- no fallback secret storage.

### OpenBao/provider unavailable
- provider automation falls back to manual operational state;
- core renewal/billing remains available.

### Chatwoot unavailable
- support handoff action disabled/degraded;
- business service state unaffected.

### Optional module absent
- integration tab/action hidden or marked unavailable;
- no fatal bootstrap errors.

## 13. Updater Architecture

Follow Magic Login hardened direction:
- semantic versions;
- GitHub Releases;
- immutable tags/releases;
- release ZIP + SHA-256;
- validate remote version > installed version;
- validate expected module root/folder;
- reject path traversal/symlinks/unexpected archive shape;
- backup current module;
- safe extraction to staging/temp;
- migration preflight;
- atomic-ish replacement per Perfex constraints;
- fail closed on checksum/archive/version mismatch;
- rollback restore on install failure where possible;
- `release_handler.php` / native Perfex integration where appropriate;
- record updater activity/audit;
- do not auto-update major/minor releases without explicit policy;
- patch-only auto-update may be considered later, never assumed.

No published tag is retargeted.

## 14. Development / Release Workflow

Project-wide development direction:
- local git / Forgejo is the primary development/testing path when available;
- Dell-hosted Perfex staging is used for local integration tests;
- GitHub is the public/release repository and may host review branches when explicitly requested;
- do not use GitHub Actions as required CI;
- release promotion occurs only after local/staging validation.

## 15. Legacy Architecture Handling

Current repository:
- metadata says version 1.0.1;
- numeric constant says 100;
- migration 101 exists;
- legacy tables are `domain_manager` and `hosting_details`;
- plaintext username/password fields exist and are rendered directly;
- purchase verification helper calls Hopperstack;
- client integration is an admin customer-profile tab, not a true portal;
- no cron/API/email-template/updater layer exists.

Target:
- keep stable module slug for upgrade continuity;
- import non-secret data into normalized schema;
- map legacy IDs for audit;
- stop exposing/writing legacy secret columns;
- migrate/discard secrets safely;
- retire old tables only after reconciliation and backup policy.
