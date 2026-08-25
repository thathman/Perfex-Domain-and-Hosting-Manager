# Perfex Domain & Hosting Manager — Test Plan

## 1. Strategy
All automated/manual validation runs locally and on the Dell-hosted Perfex staging environment. GitHub Actions is not required and must not become the primary CI path.

Levels: unit, migration/database, application-service integration, native Perfex integration, API, permissions/security, cron/concurrency, client portal, updater/release, external-adapter failure/retry and end-to-end.

## 2. Unit Tests
### Catalog
Domain normalization, hosting validation, lifecycle transitions, same-customer relations, bundle ownership and archive rules.

### Pricing
All policies: fixed, manual, percentage increase, fixed increase, cost-plus %, cost-plus fixed. Cases: missing cost, invalid values, currency precision, min price/margin, one-off override, change %, effective dates.

### Renewal
Window/invoice dates, deterministic term keys, state transitions, completed-cycle protection, payment ≠ fulfilment, new-expiry derivation, expired imports, leap-year/end-of-month cases.

### Idempotency
Same key/same payload returns prior result; same key/different payload conflicts; deterministic cron-step key; action uniqueness per cycle/step.

## 3. Migration Tests
Fresh install verifies normalized schema/default policy/no secret columns/permissions/templates.

Upgrade fixture from current repository includes domain/client/project, hosting/domain relationships, missing dates, expired rows, provider ambiguity, plaintext legacy username/password, null customer/project. Assert non-secret preservation/import report, legacy mapping IDs, no new secret storage/exposure, controlled Vault migration or explicit discard, count reconciliation and restart safety.

Migration chain must be contiguous from 101; failed migration stops upgrade; no-op slots are valid when deliberate; backup/restore is exercised.

## 4. Application Integration Tests
- Customer/project/contract contextual create and same-customer enforcement.
- Vault safe refs, absence, stale refs, unauthorized linking and no secret DTO fields.
- Invoice create/existing/failure/cancel/delete/grouped compatibility.
- Payment hook, duplicate event, reconciliation repair and final refund/cancel policy.

## 5. Renewal Scheduler Tests
### 180 days
Client/staff steps claim once; repeated cron no duplicate; missing recipient and mail retry visible.

### 60 days
Notices once; window opens; client eligibility requires published price; unresolved price blocks and raises staff action.

### 30 days
No invoice creates/sends exactly one; early invoice skips; repeated/multi-worker cron stays unique; invoice failure is retryable.

Optional later reminders run only when enabled/condition matches; service override beats global policy.

## 6. Concurrency Tests
Highest priority:
1. double-click Renew Now;
2. two tabs;
3. API same-key retry;
4. simultaneous different keys;
5. cron vs client renew;
6. two cron workers;
7. payment event during invoice persistence;
8. staff invoice vs cron;
9. price publish vs client renew.

Expected: at most one invoice/cycle, deterministic state, no duplicate side effects and clear conflict/retry responses.

## 7. Client Portal Tests
Authorization: own customer only; ID enumeration blocked; view-without-renew supported; contact mappings honored; inactive contact denied.

UX: before 60 no renew; at 60 eligible Renew Now; early invoice; existing invoice View/Pay; paid = Processing; completed = renewed/history; pricing unresolved = friendly unavailable; responsive/empty states.

Redaction: no provider cost, margin, internal notes, Vault IDs/secrets, machine-secret refs, admin audit or raw internal error.

## 8. Staff Permission Tests
Every capability is enforced in service layer, not just menu. Separate cost, pricing-manage, billing, override, audit/export rights. Direct URL/API attempts must fail correctly.

## 9. Security Tests
CSRF, XSS, SQL injection filters, IDOR, mass assignment, unsafe URLs, scope/permission intersection, rate limits, webhook signature/skew/replay, secret-redaction serializer, updater traversal/symlink/malformed ZIP/checksum/version downgrade and immutable published-tag process.

## 10. API Tests
Auth/scope, staff/contact boundary, pagination/filter/sort allowlists, invalid date/money contracts, stable errors, idempotency/409s, client customer override rejection, provider-cost serialization and event schema version.

## 11. Email / Notification Tests
Native templates exist, merge fields resolve safely, optional data degrades, client never sees provider cost/secrets, recipient rules are correct, duplicate cron doesn't duplicate send, failure is visible/retryable, Magic Login absence falls back to normal portal link.

## 12. Provider Adapter Contract Tests — Later
Connectivity, rate limit/timeout/auth/stale/partial data, duplicate webhook, retry/backoff/terminal failure, no secret logging, manual fallback, provider-action idempotency and mismatch reconciliation without silent overwrite.

## 13. Observability Tests
Correlation ID flows scheduler→invoice→event; audit exists; retry increments; dead-letter visible; replay no-ops safely; logs/metrics omit secrets.

## 14. Updater Tests
Release discovery: no/newer/same/lower/malformed version.

Artifact: missing ZIP/checksum, mismatch, wrong/nested root, traversal, unexpected file pattern, corrupt archive.

Install: backup, migration preflight/success/failure, file replacement failure, permissions, post-upgrade health and rollback.

Release discipline: never retarget published tag; increment existing versions; no force push; local/staging validation before release.

## 15. End-to-End
### E2E-1 Standard renewal
Create service/pricing/expiry → 180 notices → 60 window → 30 one invoice → payment → fulfilment-required → staff renews provider/new expiry → completed/history preserved.

### E2E-2 Early renewal
At ~55 days Renew Now creates invoice/payment; 30-day cron creates nothing; staff completes provider renewal; portal shows renewed.

### E2E-3 Pricing block
Provider cost increase requires review; at 60 portal has no price/Renew Now; staff alerted; finance publishes; eligibility activates.

### E2E-4 Renewal group
Domain+hosting compatible group → one Perfex invoice/two line refs → payment advances both → fulfilment completed individually.

### E2E-5 Legacy upgrade
Current prototype fixture with plaintext credentials → upgrade → new UI/API never exposes them → non-secret data preserved → Vault migration/discard controlled → renewal workflow succeeds.

## 16. Exit Evidence
Each phase produces local test output, migration report, manual UI/security checklist, staging notes, known limitations and release identifiers when applicable.
