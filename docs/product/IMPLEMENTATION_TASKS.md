# Perfex Domain & Hosting Manager — Implementation Tasks

This file translates Phases 0–3 into dependency-ordered coding tasks. Every task must be independently testable and map back to the product/architecture specs.

## Phase 0 — Foundation / Security Cleanup

### T0.1 Create architecture skeleton
**Depends:** none. Create application-service/repository/DTO/event structure, coding conventions and keep `domain_manager` slug. **Done when:** module boots, local smoke test passes and no business behavior changes except wiring.

### T0.2 Lock legacy credential exposure
**Depends:** T0.1. Remove/disable username/password inputs/rendering/controller writes; add serialization/render regression test. **Done when:** active UI/API has no secret field.

### T0.3 Remove legacy purchase verification
**Depends:** T0.1. Remove Hopperstack/Envato gating/warnings/external call. **Done when:** module works without license server and no call to `verify.hopperstack.com` exists.

### T0.4 Define granular permissions/authorization service
**Depends:** T0.1. Register spec capabilities; map legacy permissions conservatively where practical. **Done when:** every action has capability and direct-route tests pass.

### T0.5 Migration 102 — normalized core schema
**Depends:** T0.1. Create services/bundles/providers/type details/relations/entity associations/Vault links/indexes without destroying legacy data. **Done when:** fresh and 101→102 tests pass.

### T0.6 Structured audit service
**Depends:** T0.5. Redacted before/after, actor/origin/correlation plus native activity bridge. **Done when:** create/edit emits audit + activity without secrets.

### T0.7 Event publisher interface
**Depends:** T0.1. Namespaced envelope/local adapter/post-commit rule. **Done when:** committed transaction emits once; rollback emits none.

### T0.8 Updater/release scaffold
**Depends:** T0.1. Release discovery, semver, ZIP/checksum/archive safety, backup/preflight hooks. **Done when:** malformed/checksum/version/archive unit tests pass.

### T0.9 Local test harness
**Depends:** T0.1. Local config/fixtures including current baseline; no GitHub Actions. **Done when:** documented one-command local test path works.

## Phase 1 — Service Catalog / Workspace

### T1.1 Provider repository/service
**Depends:** T0.5,T0.4. Normalized provider CRUD/capabilities/safe metadata only. **Done when:** duplicate normalization/permission tests pass.

### T1.2 ServiceCatalogService
**Depends:** T0.5,T0.6,T0.7. Create/read/update/archive services, customer boundaries, type dispatch. **Done when:** lifecycle tests pass.

### T1.3 Domain details
**Depends:** T1.2. Domain validation, registrar/DNS/nameserver/status metadata. **Done when:** create/update tests pass.

### T1.4 Hosting details
**Depends:** T1.2. Plan/control panel/server/resource metadata. **Done when:** validation tests pass.

### T1.5 Service relations
**Depends:** T1.2. Hosting↔domain, same-customer invariant, duplicate prevention. **Done when:** one hosting safely links multiple domains.

### T1.6 Bundles
**Depends:** T1.2. Customer-scoped bundle CRUD/primary domain/attach-detach. **Done when:** customer invariant passes.

### T1.7 Perfex entity associations
**Depends:** T1.2. Project/contract roles/contextual queries. **Done when:** same-customer relationship tests pass.

### T1.8 Vault reference adapter
**Depends:** T1.2. Capability detection, link/unlink/validate safe refs; no reveal method. **Done when:** absent-Vault and permission tests pass.

### T1.9 Guided creation UI
**Depends:** T1.1–T1.8. Five steps, progressive fields, server validation, contextual prefill. **Done when:** domain/hosting E2E passes desktop/mobile.

### T1.10 Service workspace
**Depends:** T1.2–T1.8. Overview/Technical/Pricing/Billing & Renewals/Renewal Schedule/Vault/Relationships/Activity. **Done when:** authorized edits work and staff/client data boundaries hold.

### T1.11 Admin dashboard/listing
**Depends:** T1.2. Counts/search/filter/archive with safe query builder. **Done when:** filter/SQL-injection tests pass.

### T1.12 Customer/project contextual tabs
**Depends:** T1.7,T1.10. Customer `Domains & Hosting`, project `Infrastructure`, contextual add. **Done when:** queries are scoped and storage is not duplicated.

### T1.13 Migration 103 — legacy non-secret import
**Depends:** T1.2–T1.7. Map domains/hosting/providers/clients/projects, record legacy IDs, reconciliation report. **Done when:** counts reconcile and ambiguous rows are reported.

### T1.14 Legacy credential migrate/discard utility
**Depends:** T1.8,T1.13. Detect secrets; authorized Vault migration or explicit discard after backup; null only after successful verification/authorization; no logs. **Done when:** security tests pass.

## Phase 2 — Pricing / Renewal Engine

### T2.1 Migration 104 pricing/renewal schema
Create service pricing/history, policies/steps/cycles/actions, billing groups. **Done when:** migration/index/uniqueness tests pass.

### T2.2 Pricing policy engine
Implement fixed/manual/% increase/fixed increase/cost-plus %/cost-plus fixed with decimal-safe arithmetic and guardrails. **Done when:** policy matrix passes.

### T2.3 Price history/publish workflow
Preview/review/publish, immutable history, cycle snapshot. **Done when:** unresolved price cannot reach invoice path.

### T2.4 Pricing review queue
Provider-cost delta/current/proposed price/margin/accept-override-hold/permission separation. **Done when:** cost-confidentiality tests pass.

### T2.5 Seed default renewal policy
180 client+staff, 60 client+staff+open window, 30 ensure+send invoice. **Done when:** present on fresh/upgrade and editable by settings permission.

### T2.6 Renewal cycle service
Deterministic term key, state machine, snapshot/next dates. **Done when:** duplicate cycle tests pass.

### T2.7 Renewal eligibility
Window calculation, published-price gate, client/staff mode, Request Renewal alternative. **Done when:** boundary-day tests pass.

### T2.8 Renewal action/idempotency ledger
Claim/fingerprint/result/retry/correlation. **Done when:** concurrent claim permits one side-effect owner.

### T2.9 Renewal scheduler
Due-cycle selection, milestone evaluation, step claim/delegation/result. **Done when:** repeat cron produces no duplicate side effects.

### T2.10 Manual fulfilment
Processing/renewed states, actual new expiry, actor/provider ref, next-cycle derivation. **Done when:** payment alone cannot complete renewal.

### T2.11 Renewal-group logic
Compatibility checks while preserving individual cycles. **Done when:** incompatible customer/currency groups reject.

## Phase 3 — Billing / Notifications / Client Portal

### T3.1 BillingOrchestrator
Exactly-once Perfex invoice ensure, line refs, group composition/result persistence. **Done when:** concurrent creation yields one invoice.

### T3.2 Invoice/payment reconciliation
Native hooks + repair job + cancellation/deletion handling. **Done when:** duplicate/missed events converge.

### T3.3 Native email templates/merge fields
Register required keys, safe/staff-only fields and localization. **Done when:** templates render without secrets.

### T3.4 Notification executor
Staff/client email + native in-app with delivery state/retry. **Done when:** 180/60 repeated cron sends once.

### T3.5 Wire 30-day invoice actions
Ensure/send/early-invoice skip. **Done when:** standard-renewal E2E passes.

### T3.6 Client contact permissions
View/renew/notice flags/defaults/server enforcement. **Done when:** IDOR/tenant tests pass.

### T3.7 Client portal service list/detail
Cards/status/history/mobile/redaction. **Done when:** client-safe suite passes.

### T3.8 Client Renew Now
Idempotent action, early invoice, normal payment route, existing-invoice behavior. **Done when:** double-click/two-tab/cron race tests pass.

### T3.9 Paid/processing/completed portal states
Correct copy/actions/history. **Done when:** payment never falsely displays Renewed.

### T3.10 Migration 105 indexes/client access/audit hardening
**Done when:** representative query plans/performance acceptable.

### T3.11 Full MVP E2E/security gate
Run TEST_PLAN locally + staging. **Done when:** all MVP launch blockers cleared.

## Traceability

| Requirement | Tasks |
|---|---|
| Vault/no passwords | T0.2,T1.8,T1.14 |
| Modern UX | T1.9–T1.12 |
| Provider normalization | T1.1 |
| Many-domain hosting | T1.5 |
| Pricing adjustment | T2.2–T2.4 |
| 180/60/30 | T2.5,T2.9,T3.4–T3.5 |
| Client renewal from 60 days | T2.7,T3.7–T3.8 |
| Auto invoice/idempotency | T2.8,T3.1,T3.5 |
| Payment vs fulfilment | T2.10,T3.2,T3.9 |
| Renewal groups | T2.11,T3.1 |
| Audit/events | T0.6–T0.7 |
| Updater | T0.8 |
| Client permissions | T3.6 |
