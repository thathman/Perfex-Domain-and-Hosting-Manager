# Perfex Domain & Hosting Manager — Permissions & Security

## 1. Staff Permission Matrix

Recommended capabilities:

| Capability | Purpose |
|---|---|
| `domain_hosting_view` | View service inventory and safe technical metadata |
| `domain_hosting_create` | Create services/bundles |
| `domain_hosting_edit` | Edit safe service metadata |
| `domain_hosting_archive` | Archive/cancel services |
| `domain_hosting_relationships` | Attach/detach projects/contracts/services |
| `domain_hosting_pricing_view` | View client pricing |
| `domain_hosting_cost_view` | View provider cost/margin |
| `domain_hosting_pricing_manage` | Change/review/publish pricing |
| `domain_hosting_renewals_view` | View renewal cycles/history |
| `domain_hosting_renewals_manage` | Modify/complete renewal workflows |
| `domain_hosting_billing` | Generate/reissue renewal invoices |
| `domain_hosting_override` | Exceptional policy/payment override |
| `domain_hosting_providers_view` | View provider directory |
| `domain_hosting_providers_manage` | Configure provider metadata/integrations |
| `domain_hosting_vault_link` | Attach/detach Vault references |
| `domain_hosting_audit_view` | View structured audit |
| `domain_hosting_export` | Export allowed data |
| `domain_hosting_api` | Use/API-enable module endpoints subject to scopes |
| `domain_hosting_settings` | Configure global policies/templates/settings |

Do not collapse commercial/security actions into one broad `manage` capability.

## 2. Suggested Roles
Technical operator: view/create/edit/relationships/renewal operations/provider view/Vault linking; no cost or settings by default.

Account manager: inventory/client price/renewal view and limited edit; no provider cost/integration config by default.

Finance: client price/provider cost, pricing management, renewal view and billing.

Administrator: all capabilities.

## 3. Client Contact Permissions

| Permission | Meaning |
|---|---|
| `view_domain_hosting_services` | See allowed service cards/details |
| `view_domain_hosting_renewals` | See renewal dates/history |
| `renew_domain_hosting_services` | Use Renew Now/Request Renewal |
| `receive_domain_hosting_notices` | Receive renewal notices |
| invoice viewing/payment | Delegated to native Perfex invoice permissions |

Organisation Self-Service may later map roles to these permissions, but Domain Manager enforces final authorization.

## 4. Client Data Redaction
Never expose provider cost, margin, internal notes, internal integration errors, machine-secret refs, internal Vault IDs, passwords/tokens/private keys/EPP codes, correlation IDs or staff-only audit data.

Client-safe values include service name/type, intentionally exposed provider label/technical metadata, expiry/renewal date, published price, billing cycle, renewal state, invoice/payment link/status and renewal history.

## 5. Authorization Boundaries
- Every service has exactly one customer; client repository queries include authenticated `customer_id` before retrieval.
- Project/contract/service relationships must respect customer ownership.
- Provider cost/margin needs a distinct capability.
- Client renew requires open window + published price + contact permission.
- Unpaid/policy overrides require explicit override permission and audit.
- Provider integration/OpenBao refs require provider-manage.
- Domain Manager can associate Vault references; Vault separately authorizes secret access.

## 6. Browser Security
- Native Perfex CSRF for state changes.
- State-changing/destructive actions use POST/PUT/PATCH/DELETE, never GET links.
- Escape output by default; sanitize allowlisted rich text.
- Validate URL schemes/attributes.
- No credentials in query strings.
- Replace the prototype's raw request-value SQL filtering.

## 7. API Security
Shared authentication, least-privilege scopes intersected with actor permissions, idempotency for financial side effects, rate limiting, strict DTOs, allowlisted filters/sorts, no mass assignment, tenant scope in application services, token expiry/revocation and request/correlation audit.

## 8. SQL / Storage Security
Use query builder/prepared parameters. Never concatenate request values into SQL conditions. Use fixed decimal arithmetic, validate JSON metadata against allowlists and redact sensitive audit state before persistence. No secret columns.

## 9. Webhook Security
Inbound: signature validation, timestamp/skew check, replay/delivery-ID detection, idempotency, payload size/content-type/schema checks and correlation IDs. Outbound: signed delivery, stable IDs, retry/backoff, dead-letter/replay, redacted payloads.

## 10. File Security
MVP requires no arbitrary file upload. Future attachments must use Perfex file controls, MIME/extension allowlists, size limits, server-side names, authorization, malware-scanning capability where available and never accept executable module/script archives through service forms.

## 11. Audit-Sensitive Actions
Audit service create/edit/archive/customer transfer/relationships, Vault linking, provider integration changes, cost/client-price changes, price publish, bulk pricing, policy changes, client Renew Now, invoice create/retry/cancel reconciliation, payment overrides, mark-renewed/new-expiry correction, failed automation replay, export and settings/permissions changes.

## 12. Sensitive Read Logging
Consider audit for provider-cost/margin export, audit export, provider integration config reads and Vault deep-link requests. Do not duplicate Vault's secret-reveal audit.

## 13. Legacy Security Remediation
Current prototype stores and directly renders passwords, uses raw `$_GET` filter concatenation, destructive GET delete routes and legacy Hopperstack purchase verification.

Required remediation:
1. remove secret fields from active forms/controllers/views;
2. remove direct secret rendering;
3. replace unsafe filters with validated query builder;
4. use POST+CSRF for destructive/financial actions;
5. remove Hopperstack purchase-code runtime;
6. migrate/discard legacy secret material safely;
7. enforce authorization on reads and writes in application services, not just menu/UI checks.

## 14. Required Threat Tests
Client IDOR, unauthorized cost reads, duplicate Renew Now, cron/client race, replayed webhook, malicious filters, XSS, CSRF, stale/deleted invoice recovery, manipulated renewal price, Vault/OpenBao outage, scope-without-role authorization, updater path traversal/malicious archive and accidental secret serialization.

## 15. Security Acceptance
No plaintext secret storage/exposure; browser/API authorization and CSRF pass; tenant/cost-permission tests pass; SQL-injection and XSS regressions pass; duplicate financial-action tests pass; webhook replay tests pass; updater checksum/archive validation passes; required audit coverage exists.
