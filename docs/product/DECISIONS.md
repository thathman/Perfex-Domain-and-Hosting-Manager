# Perfex Domain & Hosting Manager — Architectural Decision Log

This file records settled decisions so future coding sessions do not reopen them without a newer explicit decision.

## ADR-001 — Perfex remains the primary business system
**Accepted.** Perfex owns customers, contacts, projects, contracts, invoices, payments, email templates, notifications and portal identity. Domain Manager is a native module, not a separate platform.

## ADR-002 — Module is a lifecycle system, not two CRUD lists
**Accepted.** The product manages infrastructure services, pricing and renewal lifecycle. Merely polishing Domain List/Hosting List is rejected.

## ADR-003 — Keep module slug `domain_manager`
**Accepted unless superseded by explicit migration ADR.** Display name may become Domain & Hosting Manager. Updater expects `domain_manager/`.

## ADR-004 — Guided creation, workspace after creation
**Accepted.** Wizard: type → client/relationships → details → pricing/renewal → review. Existing records use `Overview | Technical | Pricing | Billing & Renewals | Renewal Schedule | Vault | Relationships | Activity`. Giant permanent form/wizard is rejected.

## ADR-005 — Domain/Hosting are service types; hosting can serve many domains
**Accepted.** Generic service base + type details + service relations. Single `domain_id` hosting model is rejected.

## ADR-006 — Service bundles group website infrastructure
**Accepted.** Bundle can connect domain/hosting and future SSL/DNS plus project/contract/monitoring context.

## ADR-007 — Providers are normalized records
**Accepted.** Repeated free-text provider data is rejected.

## ADR-008 — Provider APIs are optional enhancements
**Accepted.** Manual mode is always first-class; core module never requires registrar/hosting API.

## ADR-009 — Vault owns human/client credentials
**Accepted.** No password fields/local credential store. Link safe Vault references only; no secret merge/API fields.

## ADR-010 — OpenBao owns machine/service secrets
**Accepted.** Provider integrations store opaque broker references only.

## ADR-011 — Default renewal schedule is 180 / 60 / 30
**Accepted.** 180 days staff+client notice; 60 days staff+client notice and client renewal opens; 30 days automatic Perfex invoice create/send if no invoice exists. This supersedes treating the requested reminders as manual staff reminders. Native manual reminders may exist later but are not the core feature.

## ADR-012 — Client can renew from portal from 60 days by default
**Accepted.** No invoice → Renew Now creates one immediately; existing unpaid invoice → View/Pay; paid → Processing; completed → renewed/history. Thirty-day scheduler must safely skip an early invoice.

## ADR-013 — Renewal invoice creation is idempotent
**Accepted.** Cron/API/staff/client share one BillingOrchestrator/action ledger. Independent invoice code paths are rejected.

## ADR-014 — Payment does not equal provider renewal
**Accepted.** Payment moves cycle to paid/fulfilment-required. MVP uses manual authorized provider completion + actual new expiry; provider automation later.

## ADR-015 — Provider cost and client price are separate/historical
**Accepted.** One mutable price field is rejected. Margin visibility is permissioned and cycle snapshots are immutable.

## ADR-016 — Price adjustment is first-class
**Accepted.** Fixed, manual review, percentage increase, fixed increase, cost-plus percentage, cost-plus fixed and one-off next-renewal override; minimum price/margin guardrails.

## ADR-017 — Pricing must be published before self-renewal/invoicing
**Accepted.** Unresolved price blocks portal renewal and scheduled invoice. Blind stale-price invoice generation is rejected.

## ADR-018 — Renewal and price history are immutable snapshots
**Accepted.** Overwriting expiry/price and losing previous-cycle history is rejected.

## ADR-019 — Renewal groups can consolidate invoices
**Accepted.** Compatible services may share one Perfex invoice while each renewal cycle keeps a distinct line/reference and fulfilment state.

## ADR-020 — Perfex invoices/payments are source of truth
**Accepted.** No competing custom payment ledger.

## ADR-021 — Native Perfex email templates own content
**Accepted.** Hard-coded bodies are prohibited; client merge fields never expose secrets/provider cost.

## ADR-022 — Client portal is purpose-built
**Accepted.** Cards/status/actions/history. Exposing admin DataTables is rejected.

## ADR-023 — Client permissions are contact-aware
**Accepted.** View and renew are separate, server-side customer/contact scoped.

## ADR-024 — Project/customer views are contextual
**Accepted.** Customer `Domains & Hosting`; project `Infrastructure`; same service layer underneath.

## ADR-025 — Forms owns generic structured collection
**Accepted.** Domain Manager does not become a form builder. Secrets never flow through ordinary Forms fields.

## ADR-026 — Onboarding owns orchestration
**Accepted.** Domain Manager exposes actions/checks/events/deep links; no separate onboarding engine.

## ADR-027 — Chatwoot owns new support communication
**Accepted.** Domain Manager provides context/human-handoff links; no competing support/chat engine.

## ADR-028 — AI/MCP consumes permission-aware contracts
**Accepted.** No AI direct DB/security bypass.

## ADR-029 — Property Monitor/Uptime Kuma own monitoring observations
**Accepted.** Domain Manager owns managed-service/business lifecycle, not a duplicate monitor engine.

## ADR-030 — Approval Workflow may own approvals
**Accepted future integration.** Price exceptions, bulk adjustments and overrides use shared approvals rather than an embedded generic approval engine.

## ADR-031 — API is designed from the beginning
**Accepted.** Internal services and public API are separate; UI is not the only business-rule implementation.

## ADR-032 — Events before tight cross-module coupling
**Accepted.** Namespaced `domain_hosting.*` events enable optional integrations.

## ADR-033 — Shared integration runtime where available
**Accepted.** Reuse webhook verification, idempotency, retry/backoff, dead-letter/replay and integration logs; bespoke fragile runtimes are rejected.

## ADR-034 — Structured audit required
**Accepted.** Financial/renewal/pricing/policy/security-sensitive actions record actor/action/resource/before-after/correlation with secret redaction.

## ADR-035 — Safe updater follows Magic Login hardened direction
**Accepted.** Semantic versions, GitHub Releases, ZIP + SHA-256, archive/version validation, backup, migration preflight, rollback considerations and immutable published tags/releases. Never retarget published versions.

## ADR-036 — Local-first testing; no required GitHub Actions
**Accepted.** Tests run locally/Dell staging; Forgejo/local git is intended project development path when available.

## ADR-037 — Current repository is not assumed greenfield
**Accepted conservative planning decision.** No explicit prior decision establishes that this specific module has never been installed, so preserve non-secret data safely rather than invent destructive compatibility assumptions. Bad architecture is still replaced. A later explicit no-install decision may simplify migrations through a new ADR.

## ADR-038 — Legacy plaintext credentials are a security migration, not compatibility
**Accepted.** New UI/API immediately stops exposure/write. Existing values are either migrated through Vault with authorization or explicitly discarded after backup. Copying them into new tables is rejected.

## ADR-039 — SSL is full vision, not MVP blocker
**Accepted.** MVP preserves service-type extension point; certificate management follows core domain/hosting lifecycle.

## ADR-040 — README distinguishes implemented/planned/future
**Accepted.** Roadmap capability cannot be advertised as already built.
