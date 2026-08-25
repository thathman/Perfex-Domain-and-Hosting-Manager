# Perfex Domain & Hosting Manager — Data Model

**Status:** Proposed implementation contract  
**Database prefix:** all names below are shown without Perfex `db_prefix()`.

## 1. Modeling Rules

- Perfex owns customers, contacts, projects, contracts, invoices, payments, tasks and subscriptions.
- Domain Manager stores stable references to those records.
- New schema stores no plaintext passwords, API tokens, private keys or transfer/EPP codes.
- Monetary values use fixed decimal types.
- Historical price/renewal rows are append-oriented.
- Soft archive is preferred for business resources; hard delete is limited to safe configuration rows with no dependent history.
- Cross-module foreign keys are primarily logical because Perfex/module installation order and table constraints vary; enforce referential integrity in services and indexes.
- Use UTC timestamps for application timestamps; business renewal dates remain DATE in the relevant business timezone semantics.
- Migration numbering continues contiguously from current migration `101`.

## 2. Core Tables

### 2.1 `domain_manager_service_bundles`
Logical website/application/infrastructure grouping.

Key columns: `id BIGINT`, `customer_id INT NOT NULL`, `name VARCHAR(191)`, optional `slug`, optional `primary_domain_service_id`, `status`, `description`, `created_by`, timestamps, archive actor/timestamp.

Indexes: `(customer_id,status)`, `(customer_id,name)`.

### 2.2 `domain_manager_services`
Base service record.

Key columns:
- `id BIGINT UNSIGNED PK AI`
- `customer_id INT NOT NULL`
- optional `bundle_id`, `provider_id`
- `service_type VARCHAR(32)` — domain, hosting; future ssl/dns/server/email
- `name VARCHAR(191)`
- `lifecycle_status VARCHAR(32)` — active, suspended, cancelled, expired, archived
- `management_status VARCHAR(32)` — managed, client_managed, external, unknown
- `managed_by_us BOOLEAN`
- `assigned_staff_id INT NULL`
- `start_date DATE NULL`
- `current_expiry_date DATE NULL`
- `auto_renew_external BOOLEAN`
- `renewal_enabled BOOLEAN`
- `client_self_renew_enabled BOOLEAN`
- optional `renewal_policy_id`, `billing_group_id`
- `notes_internal TEXT NULL`
- legacy trace columns `legacy_source_type`, `legacy_source_id`
- created/updated/archive metadata.

Indexes: `(customer_id,service_type,lifecycle_status)`, `(current_expiry_date,renewal_enabled,lifecycle_status)`, `(provider_id,lifecycle_status)`, `(bundle_id)` and unique legacy mapping when non-null.

### 2.3 `domain_manager_domain_details`
One-to-one where `service_type=domain`.

Columns: `service_id PK`, canonical `domain_name VARCHAR(253)`, optional registrar provider, registration date/status, DNS provider, safe nameserver JSON, privacy flag, transfer status metadata, WHOIS status, `updated_at`.

Canonical domain name is unique unless a later ADR defines an alias/multi-customer exception.

### 2.4 `domain_manager_hosting_details`
One-to-one where `service_type=hosting`.

Columns: `service_id PK`, plan name, non-secret provider account reference, control-panel type and safe URL, server hostname/IP, allowlisted resource metadata JSON, environment and `updated_at`.

A hosting service is related to domains via `domain_manager_service_relations`; it does not own a single `domain_id`.

### 2.5 `domain_manager_providers`
Normalized provider directory.

Columns: `id`, `name`, `provider_type`, website/support URLs, capability metadata, adapter key, opaque `machine_secret_reference`, status and timestamps. The reference points to OpenBao/shared broker; no secret is stored here.

## 3. Relationship Tables

### 3.1 `domain_manager_service_relations`
`id`, source service, target service, `relation_type` (`hosts_domain`, `uses_dns`, `secures`, `replaces`, `depends_on`), safe metadata, creator/timestamp.

Unique `(source_service_id,target_service_id,relation_type)`. Both services must share a customer in MVP.

### 3.2 `domain_manager_entity_associations`
References Perfex/module-owned objects: `service_id`, `entity_type` (`project`, `contract`, `task`, `estimate`, `proposal`, `subscription`, `property`), `entity_id`, optional role, creator/timestamp.

Customer ownership remains `services.customer_id` rather than a generic association.

### 3.3 `domain_manager_contact_access`
Optional per-contact policy: service/contact, `can_view`, `can_renew`, `receive_renewal_notices`, timestamps. Unique `(service_id,contact_id)`.

### 3.4 `domain_manager_vault_links`
Safe references only: resource type/id, opaque Vault reference ID, safe display label, creator/timestamp. No cached secret values.

## 4. Pricing Tables

### 4.1 `domain_manager_service_pricing`
One current configuration per service:
- currency CHAR(3)
- billing cycle
- provider cost DECIMAL(20,4)
- client price DECIMAL(20,4)
- policy type: fixed, manual, percent_increase, fixed_increase, cost_plus_percent, cost_plus_fixed
- policy value
- minimum client price/margin amount/margin percent
- review state, reviewer/timestamp
- effective date/update timestamp.

### 4.2 `domain_manager_price_history`
Append-oriented history with service, optional renewal-cycle ID, currency, provider cost, client price, margin snapshots, reason, policy snapshot, effective period, creator/timestamp.

Indexes `(service_id,effective_from)`, `(renewal_cycle_id)`. Historical rows are not silently rewritten.

## 5. Renewal Tables

### 5.1 `domain_manager_renewal_policies`
Header with scope, default flag, `self_renew_open_days DEFAULT 60`, `invoice_create_days DEFAULT 30`, status and timestamps. Only one active global default.

### 5.2 `domain_manager_renewal_policy_steps`
Policy milestones: policy ID, `days_before_expiry`, `action_type`, optional Perfex template key, allowlisted conditions, sequence, enabled flag.

Seed default steps:
- 180 client email;
- 180 staff email;
- 60 client email;
- 60 staff email;
- 60 open window;
- 30 ensure invoice;
- 30 send invoice.

Unique step identity prevents duplicate policy actions.

### 5.3 `domain_manager_renewal_cycles`
One cycle per service/term:
- `service_id`, deterministic `term_key`
- previous expiry, target start/expiry
- renewal-window open and scheduled invoice dates
- currency/provider-cost/client-price snapshots
- `pricing_status` pending/review_required/published
- state: upcoming, window_open, invoiced, awaiting_payment, paid, fulfilment_required, processing, renewed, declined, cancelled, expired, failed
- Perfex `invoice_id` and stable line reference
- payment timestamp
- provider-renewal timestamp/reference
- actual new expiry
- completion actor/timestamps.

Unique `(service_id,term_key)` and invoice-line reference where non-null. Indexed for due-date/state and invoice lookup.

### 5.4 `domain_manager_renewal_actions`
Idempotent action ledger: cycle, optional policy step, action type, unique idempotency key, request fingerprint, state (`claimed`, `succeeded`, `failed_retryable`, `failed_terminal`, `skipped`), attempt/retry timing, safe result/error refs, correlation ID, actor/origin and timestamps.

### 5.5 `domain_manager_billing_groups`
Customer-scoped group with name, optional currency, status and consolidation strategy.

### 5.6 `domain_manager_billing_group_members`
Group/service/sequence. Members must share customer; invoice consolidation only occurs when currency/tax/due-date policy is compatible.

## 6. Audit / Integration Tables

### `domain_manager_audit_log`
Structured audit columns: actor type/id, action, resource type/id, customer ID, redacted safe previous/result JSON, correlation/request IDs, origin, optional IP/user-agent hash, timestamp.

Indexes by resource/time, customer/time and correlation ID. Never include secret values.

Integration/webhook delivery storage should use the shared Perfex ecosystem integration runtime. Do not create bespoke retry/dead-letter tables when a shared equivalent exists.

## 7. Status Definitions

Service lifecycle: `active`, `suspended`, `cancelled`, `expired`, `archived`.

Pricing: `pending`, `review_required`, `published`.

Renewal: `upcoming`, `window_open`, `invoiced`, `awaiting_payment`, `paid`, `fulfilment_required`, `processing`, `renewed`, `declined`, `cancelled`, `expired`, `failed`.

Persist meaningful lifecycle state; do not derive the whole workflow from display-time date coloring.

## 8. Legacy Tables & Upgrade Mapping

Current legacy tables: `domain_manager`, `hosting_details`. They contain plaintext username/password fields.

Upgrade rules:
1. Back up module/database per updater policy.
2. Migration 102 creates new schema without destroying legacy data.
3. Disable active UI/API reads/writes of legacy credential fields.
4. Import non-secret domain/hosting data into new services/details/associations.
5. Preserve legacy source type/ID for audit/reconciliation.
6. Reconcile counts and create an import report.
7. If legacy secrets exist: migrate them only through an authorized Vault migration utility and verify success before nulling, or explicitly discard after backup. If Vault is absent, the new app still must not expose/write the old values.
8. Legacy tables may remain read-only for a release then be removed/archived by a later contiguous migration.

No secret is copied into new Domain Manager tables or migration logs.

## 9. Migration Sequence Proposal

Existing: `101_version_101.php`.

Planned:
- `102_version_102.php` — normalized core schema, non-destructive.
- `103_version_103.php` — non-secret legacy import/mapping and legacy exposure lock metadata.
- `104_version_104.php` — pricing/renewal/action/billing group schema.
- `105_version_105.php` — contact access/audit/index support.
- `106_version_106.php` — legacy cleanup only when prerequisites are satisfied; use a deliberate no-op if cleanup is deferred so numbering stays contiguous.

Exact grouping may change during implementation; numbering may not skip.

## 10. Retention

- Services/bundles: archive rather than delete; retain with business history.
- Renewal cycles/price history: retain per financial/business policy.
- Audit: retain per ecosystem audit policy.
- Integration/retry logs: finite retention appropriate to severity/compliance.
- Idempotency records for financial side effects remain attached to cycles long enough to prevent duplicates.
- Legacy secret material is nulled/removed as soon as controlled migration/discard succeeds.

## 11. Data Integrity Invariants

- Service customer changes only through an explicit controlled transfer.
- Bundle/service customers match.
- Service relations are same-customer in MVP.
- Contact access contact belongs to service customer.
- Renewal term key is unique per service.
- Published cycle has currency + client price.
- Invoice generation requires published pricing.
- Completed renewal has actual new expiry + completion timestamp.
- Price history is immutable except controlled correction with audit.
- No active schema column stores password/token/secret material.
