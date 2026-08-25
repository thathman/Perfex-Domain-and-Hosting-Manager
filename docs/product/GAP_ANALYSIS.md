# Perfex Domain & Hosting Manager — Codebase Gap Analysis

**Repository:** `thathman/Perfex-Domain-and-Hosting-Manager`  
**Inspected baseline:** `main` @ `3df4e8ea3514c76712a642a7e693cd97d8ad56b6`  
**Inspection date:** 2026-08-25

## 1. Current Snapshot
Small imported Perfex module prototype: one main branch/two commits at inspection, no GitHub Releases/open issues, metadata `Version: 1.0.1`, numeric constant `100`, migration `101`, title-only README, separate domain/hosting CRUD, admin customer/project tabs, no true client portal, renewal jobs, email-template registration, REST API, webhook/event contract, updater, pricing/renewal history or invoice/payment integration. Plaintext username/password fields exist and are rendered.

## 2. KEEP
- Module slug/basic Perfex package — useful for install/updater continuity.
- Native hook/menu/tab registration concept — correct direction.
- Capability registration concept — replace coarse keys, retain permission-based architecture.
- Customer/project relationship intent — migrate to normalized associations.
- Valid non-secret expiry/purchase/start/provider metadata — migrate safely.

## 3. IMPROVE
- Admin navigation → operational dashboard + Services/Domains/Hosting/Renewals/Pricing/Providers.
- Domain/hosting metadata → validated type-specific records + normalized providers.
- Expiry coloring → keep as visual urgency only; lifecycle engine owns actions.
- Customer/project tabs → contextual summaries/actions/workspace links.
- `deleted` intent → real archive lifecycle instead of routine hard delete.

## 4. REFACTOR
- Controller mixes validation/persistence/license logic → thin controllers + application services.
- God-ish CRUD models → focused repositories/services.
- Giant forms → guided create + workspace.
- Inconsistent manual statuses → defined state machine.
- Raw `$_GET` filters → validated query-builder filters.
- GET delete links → POST+CSRF archive/delete policy.
- Repeated provider name/URL → normalized provider model.

## 5. REPLACE
- Legacy two-table architecture — cannot support bundles, many-domain hosting, price history, renewal cycles, self-service or extensible service types cleanly.
- `hosting_details.domain_id` one-domain model → service relations.
- Credential fields/storage/rendering → Vault/OpenBao references only.
- Hopperstack/Envato purchase verification → remove.
- Admin-table-style customer surface → purpose-built client portal.

## 6. REMOVE
- Hopperstack verification helper/runtime.
- Envato purchase-code settings and gating flags.
- active username/password inputs and rendering.
- fake `https://default-url.com` fallback.
- duplicated bootstrap line in client tab.
- hard-coded duplicate status lists.
- hard delete as normal service lifecycle.
- raw request concatenation in DataTable filtering.
- obsolete commented code after migration.

## 7. NEW
Application services/DTOs/events/audit/idempotency/updater/local tests; normalized services/bundles/providers/relations/contact access/Vault refs/pricing/history/renewal policies/cycles/actions/billing groups; renewal scheduler; 180/60/30 policy; client renewal from 60 days; early and scheduled idempotent Perfex invoice generation; payment/fulfilment separation; pricing adjustment policies/review gate; renewal groups; true portal; safe merge fields; API v1; signed webhooks; integration diagnostics and cross-module contracts.

## 8. File-Level Assessment

| Current path | Classification | Action |
|---|---|---|
| `README.md` | REPLACE | accurate current/planned/future README |
| `domain_manager/domain_manager.php` | REFACTOR | retain bootstrap role; remove license warning; add hooks/permissions/templates/jobs/updater |
| `domain_manager/controllers/Domain_manager.php` | REPLACE/REFACTOR | focused thin controllers |
| `domain_manager/helpers/domain_manager_helper.php` | REMOVE/REPLACE | remove Hopperstack verification |
| `domain_manager/install.php` | REFACTOR | delegate install to contiguous migrations/new schema |
| `domain_manager/migrations/101_version_101.php` | KEEP HISTORIC | never rewrite; continue at 102 |
| `domain_manager/models/Domain_manager_model.php` | REPLACE | repositories/service catalog |
| `domain_manager/models/Hosting_details_model.php` | REPLACE | hosting details + service relations |
| `domain_manager/views/create.php` | REPLACE | guided wizard |
| `domain_manager/views/edit.php` | REPLACE | workspace editing |
| `domain_manager/views/view.php` | REPLACE | no-secret service workspace |
| `domain_manager/views/hosting/*` | REPLACE | unified service architecture |
| `domain_manager/views/tables/*` | REFACTOR/REPLACE | safe filters/operational queues |
| admin customer/project views | REFACTOR | contextual summaries |
| `domain_manager/views/manage.php` | REPLACE | policies/providers/updater settings |
| language strings | IMPROVE | full localization; remove hard-coded English |
| `docs/product/*` | NEW | spec kit |
| `specs/*` | NEW | implementation feature contracts |

## 9. Security Gaps
Critical: plaintext password storage/rendering, raw request filter concatenation, destructive GET deletes, inconsistent read/write authorization and unnecessary external license call.

High: no immutable financial/renewal audit, no side-effect idempotency, no tenant-scoped client authorization layer and no secret-redaction API serializer.

## 10. Product Gap Matrix

| Capability | Current | Target |
|---|---|---|
| Domain/Hosting CRUD | yes | service lifecycle/workspace |
| many domains per hosting | no | yes |
| provider normalization/bundles | no | yes |
| pricing/cost/history | no | yes |
| renewal schedule/history | no | yes |
| 180 notice / 60 renew / 30 invoice | no | yes |
| Perfex invoice/payment integration | no | yes |
| renewal groups | no | yes |
| real client portal | no | yes |
| native email templates | no | yes |
| API/events/webhooks | no | yes |
| structured audit | no | yes |
| Vault links | no | yes |
| OpenBao/provider adapters | no | later |
| updater | no | yes |
| SSL/monitoring context | no | later |

## 11. Reuse Recommendation
Reuse slug/path, historical migration untouched, current-column knowledge for import mapping, native customer/project hook intent and safe legacy data. Do not preserve current controller/model/view architecture simply to reduce rewrite volume.

## 12. Compatibility Recommendation
No recovered explicit decision says this specific module is definitely unused/greenfield. Therefore planning is conservative: keep 101, create 102+, import non-secret data, handle legacy credentials through controlled Vault migration/discard, then retire legacy tables. If an explicit later decision confirms zero installs/data, simplify through a new ADR rather than silently assuming it.
