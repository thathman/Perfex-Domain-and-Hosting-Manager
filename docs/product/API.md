# Perfex Domain & Hosting Manager — API Contract

**Status:** Design contract  
**Version:** v1  
**Base path (planned):** `/api/v1/domain-hosting`

The exact Perfex API routing/bootstrap mechanism may adapt to the project-wide shared API layer. Resource/action semantics in this document are stable unless superseded by ADR.

## 1. Principles

- API uses the same application services and authorization as admin/client UI.
- Public API exposure and internal service contracts are separate.
- Secrets never appear in request/response bodies.
- All customer-scoped queries enforce customer boundary before retrieval.
- Side-effect actions use idempotency keys where duplicate execution is harmful.
- Pagination/filtering are first-class.
- Webhooks are signed, replay-protected and auditable.
- Errors use stable machine-readable codes.

## 2. Authentication
Use the shared Perfex/API authentication mechanism selected for the wider project: authenticated principal, scopes, actor context, token expiry/revocation. Do not build a custom long-lived unaudited token table in this module.

## 3. Scopes
- `domain_hosting.read`
- `domain_hosting.write`
- `domain_hosting.relationships`
- `domain_hosting.pricing.read`
- `domain_hosting.pricing.manage`
- `domain_hosting.renewals.read`
- `domain_hosting.renewals.manage`
- `domain_hosting.billing`
- `domain_hosting.providers.read`
- `domain_hosting.providers.manage`
- `domain_hosting.audit.read`
- `domain_hosting.client_self_service`
- `domain_hosting.export`
- `domain_hosting.webhooks.manage`

A token scope never grants more than the actor's underlying Perfex/module permission.

## 4. Common Envelopes
Success uses `data` and `meta.request_id`. Collections also expose page/per_page/total and next/prev links. Errors use `error.code`, safe `error.message`, optional validation `details`, and request ID. No stack traces or secret/internal configuration details.

## 5. Pagination / Filtering
Default `per_page=25`, maximum 100. Allowlisted filters include customer, service type/status, provider, project, bundle, renewal range/state, pricing-review state and search. Sort fields are allowlisted.

## 6. Service Resources
- `GET /services` — list; scope `domain_hosting.read`.
- `POST /services` — create; scope `domain_hosting.write`; retrying callers use `Idempotency-Key`.
- `GET /services/{id}` — safe detail.
- `PATCH /services/{id}` — update allowed metadata; customer transfer/type changes use controlled commands.
- `POST /services/{id}/archive` — archive.
- `POST /services/{id}/relationships` — attach project/contract/bundle/service relationship.
- `DELETE /services/{id}/relationships/{relationship_id}` — detach.

Service payloads include stable IDs, type, name, customer/provider/bundle refs, lifecycle status and safe renewal summary. Provider cost/margin is omitted unless both scope and staff permission allow it.

## 7. Bundles
- `GET /bundles`
- `POST /bundles`
- `GET /bundles/{id}`
- `PATCH /bundles/{id}`
- `POST /bundles/{id}/services/{service_id}`

Bundle and service customers must match.

## 8. Providers
- `GET /providers`
- `POST /providers` with `domain_hosting.providers.manage`.

Provider payload contains safe metadata/capability keys and at most an opaque OpenBao/shared-broker machine-secret reference. Secret values are never accepted or returned.

## 9. Pricing
- `GET /services/{id}/pricing`
- `POST /services/{id}/pricing/preview`
- `POST /services/{id}/pricing/publish`
- `POST /pricing/bulk-preview`
- `POST /pricing/bulk-apply` (permission/approval gated; may be deferred beyond MVP execution).

Preview returns current/proposed client price, provider cost only if allowed, margin and warnings. Publish is idempotent and records history/audit.

## 10. Renewals
- `GET /renewals`
- `GET /renewals/{id}`
- `POST /services/{id}/renewals/ensure` — internal/admin idempotent ensure.
- `GET /services/{id}/renewal-eligibility`
- `POST /renewals/{id}/renew` — client/admin early renewal; requires idempotency key.
- `POST /renewals/{id}/invoice` — staff explicit invoice request using the same BillingOrchestrator path as cron/client.
- `POST /renewals/{id}/complete` — records actual provider-side completion/new expiry.
- `POST /renewals/{id}/retry` — retry safe failed step.

`renew` validates renewal window and published pricing, ensures exactly one invoice, and returns created/existing invoice reference/payment route. It never marks provider fulfilment complete.

## 11. Client API
Where the shared API supports contact authentication:
- `GET /client/services`
- `GET /client/services/{id}`
- `GET /client/renewals`
- `POST /client/renewals/{id}/renew`

Customer context is derived from authenticated contact; arbitrary client-supplied `customer_id` cannot escape tenant scope.

## 12. Audit API
`GET /audit` with `domain_hosting.audit.read`, filterable by resource, actor, action, correlation ID and date. Sensitive before/after values are redacted.

## 13. Internal Service Contracts
HTTP is not required for module-to-module calls. Recommended interfaces:

```php
interface DomainHostingCatalog {
    public function getService(int $id, ActorContext $actor): ServiceDto;
    public function listServices(ServiceFilter $filter, ActorContext $actor): Page;
}

interface DomainHostingRenewals {
    public function getEligibility(int $serviceId, ActorContext $actor): RenewalEligibilityDto;
    public function renew(int $renewalId, RenewalCommand $command, ActorContext $actor): RenewalResult;
}

interface DomainHostingAssociations {
    public function attachProject(int $serviceId, int $projectId, string $role, ActorContext $actor): void;
}
```

MCP/Ara, Handoff, Onboarding and other modules prefer these contracts/events over direct SQL.

## 14. Idempotency
Required for service creation by retrying external callers, invoice ensure/create, client renew, bulk price apply, future provider renewal and replayed commands/webhooks.

Store actor/scope/action/key hash, request fingerprint and result reference. Reuse of one key with a different request returns `DH_CONFLICT`.

## 15. Rate Limiting
Use shared API gateway/runtime. Reads may allow moderate volume; mutations lower; client renewal very low burst and idempotent; provider/admin config low. Rate-limit rejection does not consume an idempotency key.

## 16. Webhooks
Use shared delivery runtime. Outbound deliveries are signed and contain timestamp, delivery ID and event ID. Receivers validate signature, clock skew and replay ID. Suggested headers: `X-Perfex-Event`, `X-Perfex-Delivery`, `X-Perfex-Timestamp`, `X-Perfex-Signature`.

## 17. Webhook Events
Externally subscribable core events:
- `domain_hosting.service.created`
- `domain_hosting.service.updated`
- `domain_hosting.price.review_required`
- `domain_hosting.price.published`
- `domain_hosting.renewal.window_opened`
- `domain_hosting.renewal.invoice_created`
- `domain_hosting.renewal.payment_received`
- `domain_hosting.renewal.fulfilment_required`
- `domain_hosting.renewal.completed`
- `domain_hosting.renewal.failed`
- `domain_hosting.service.expired`

## 18. Versioning
HTTP uses `/v1`; event envelopes contain `event_version`. Additive changes remain v1; breaking semantics require v2. Internal DTO/interfaces use explicit versions/adapters when necessary.

## 19. Validation
Validate normalized/punycode-safe domains, date relationships, customer boundaries, provider/entity existence, decimal-string money, currency, price-policy parameters, safe URLs and forward renewal dates unless an authorized correction. Generic metadata must reject secret-like keys/values.

## 20. Audit Requirements
Every mutation records actor, origin (`admin`, `client_portal`, `api`, `cron`, `integration`, `migration`), request/correlation ID, idempotency-key hash where relevant, resource/action, safe before/after summary, timestamp and interactive IP/device metadata where appropriate.
