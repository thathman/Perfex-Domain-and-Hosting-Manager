# Perfex Domain & Hosting Manager — Cross-Module Review

This review records the ownership check before implementation so future coding sessions do not recreate capabilities owned elsewhere.

## Ownership Questions

### Service inventory / renewal lifecycle
**Domain & Hosting Manager owns it.** No other current Perfex module owns managed domain/hosting service inventory, pricing policy or renewal-cycle orchestration.

### Customers / Contacts / Projects / Contracts / Invoices / Payments
**Perfex core owns them.** Domain Manager stores stable references and orchestrates meaningful service relationships; it does not duplicate these records.

### Credentials
**Vault owns human/client credentials. OpenBao owns machine/service secrets.** Domain Manager stores references only.

### Forms
**Perfex Forms owns generic collection.** The service creation wizard is an internal business-object workflow, not a replacement form builder. External requests should use Forms.

### Onboarding
**Perfex Onboarding owns orchestration/state.** Domain Manager exposes checks, actions, events and deep links.

### Support / chat
**Chatwoot owns new support communication.** Domain Manager only provides contextual service/renewal data and human-handoff actions.

### Monitoring
**Property Monitor/Uptime Kuma own operational observations.** Domain Manager owns service identity and commercial lifecycle, not raw monitor history.

### Approvals
**Approval Workflow owns approval state.** Domain Manager can request/consume approval for pricing/override cases.

### AI
**MCP/Ara owns AI orchestration.** Domain Manager exposes structured permission-aware resources/actions; no embedded AI bypass.

## Reusable Platform Capabilities
Prefer shared implementations for event/webhook runtime, API auth/scopes, idempotency/retry/dead-letter, structured audit, notification inbox, approvals, OpenBao secret broker and canonical identity mapping. Domain Manager should depend on ports/adapters so shared platform components can replace temporary local fallbacks.

## Event vs Tight Coupling
Use events for service/bundle changes, relationship changes, price review/publish, renewal-window opening, invoice success/failure, payment, fulfilment requirement, renewal completion/failure, expiry and integration failures. Use synchronous calls only when an immediate result is required, such as invoice creation or validating a Vault reference.

## API / MCP Need
Yes. Other modules need customer/project service lookup, bundle lookup, renewal eligibility, attach/detach association commands, safe pricing/renewal summary and renewal actions. MCP/Ara uses exactly these services and never direct SQL.

## Native Perfex Placement
- **Customers:** service inventory, upcoming renewals, invoice/renewal state and contextual add action.
- **Contacts:** permissions/notification recipients; no duplicate contact store.
- **Projects:** `Infrastructure` tab and relationships.
- **Contracts:** relationship and structured commercial-rule metadata only.
- **Invoices:** service/renewal metadata links back to cycle; invoice remains Perfex-owned.
- **Properties:** optional association via Property Monitor/extensions.
- **Tasks:** optional operational follow-up links; no task engine.

## Cross-Cutting Needs
- Merge fields: yes, safe service/renewal/pricing fields; provider cost staff-only; secrets forbidden.
- Notifications: yes, renewal/pricing/invoice/payment/failure events.
- Email templates: yes, native Perfex; no hard-coded bodies.
- Cron: yes, renewal scheduler, payment reconciliation fallback, retry worker, future provider sync.
- Webhooks: yes, shared runtime where possible.
- Audit: yes, especially financial/pricing/renewal/policy/security-sensitive actions.
- Client portal: yes; self-service renewal from 60 days is core.

## Ownership Summary

| Concern | Owner |
|---|---|
| Customer/contact | Perfex core |
| Project | Perfex core |
| Contract | Perfex core |
| Invoice/payment | Perfex core |
| Domain/hosting service inventory | Domain & Hosting Manager |
| Renewal policy/cycle | Domain & Hosting Manager |
| Client/provider pricing history | Domain & Hosting Manager |
| Client credentials | Vault |
| Provider machine secrets | OpenBao |
| Support conversations | Chatwoot |
| Onboarding state | Onboarding |
| Structured external form intake | Forms |
| Monitoring measurements | Property Monitor/Uptime Kuma |
| Approval state | Approval Workflow |
| AI orchestration | MCP/Ara |
| Notification aggregation | Notifications Inbox |
