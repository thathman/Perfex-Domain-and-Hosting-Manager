# Perfex Domain & Hosting Manager — Dependency Map

## 1. Required for MVP

```text
Perfex CRM
 ├─ Customers
 ├─ Contacts / Client Portal
 ├─ Projects
 ├─ Contracts
 ├─ Invoices
 ├─ Payments
 ├─ Email Templates
 ├─ Notifications
 ├─ Permissions / Roles
 ├─ Activity Logs
 └─ Cron
        |
        v
Perfex Domain & Hosting Manager
```

These are native Perfex capabilities, not separate module install dependencies.

## 2. Optional Module Integrations

```text
Domain & Hosting Manager
 ├─> Vault                     [credential references]
 ├─> Handoff                   [project infrastructure association]
 ├─> Onboarding                [actions/checks/deep links]
 ├─> Forms                     [structured requests]
 ├─> Property Monitor          [health context]
 ├─> Uptime Kuma               [monitor context]
 ├─> Chatwoot                  [support context/human handoff]
 ├─> Notifications Inbox       [event aggregation]
 ├─> Approval Workflow         [pricing/override approvals]
 ├─> Organisation Self-Service [delegated client permissions]
 ├─> Magic Login               [secure portal deep links]
 ├─> Verified Receipts/Bachs   [payment context via Perfex]
 ├─> MCP / Ara                 [permission-aware tools]
 └─> OpenBao                   [provider machine-secret references]
```

All are capability-detected and fail gracefully when absent.

## 3. Provider / Consumer Direction

Domain Manager provides service inventory, customer/project infrastructure summaries, upcoming renewal/state, authorized published pricing, safe provider/domain/hosting metadata and events.

Likely consumers: Handoff, Onboarding, Property Monitor, Chatwoot, MCP/Ara, Organisation Self-Service and external API/webhook consumers.

Domain Manager consumes customer/contact/project/contract identity, invoice/payment state, native templates/notifications/cron, Vault refs, OpenBao-brokered provider execution, approval results, property health and optional Magic Login link generation.

## 4. Event Relationships

| Event | Typical consumers |
|---|---|
| `domain_hosting.service.created` | Onboarding, Property Monitor, Handoff, MCP |
| `domain_hosting.service.updated` | Property Monitor, Handoff, MCP |
| `domain_hosting.relationship.created` | Handoff/project context |
| `domain_hosting.price.review_required` | Approval Workflow, Notifications Inbox, MCP |
| `domain_hosting.price.published` | Client portal, Notifications Inbox |
| `domain_hosting.renewal.window_opened` | Client portal, Notifications Inbox, optional Chatwoot |
| `domain_hosting.renewal.invoice_created` | Notifications Inbox, optional Chatwoot |
| `domain_hosting.renewal.payment_received` | Operations/Notifications Inbox |
| `domain_hosting.renewal.fulfilment_required` | Operations, optional Approval |
| `domain_hosting.renewal.completed` | Property/Handoff context, Notifications Inbox, MCP |
| `domain_hosting.service.expired` | Notifications Inbox, Property Monitor |
| `domain_hosting.integration.failed` | Notifications Inbox/admin observability |

## 5. API Relationships
Same-runtime modules prefer internal catalog/association/renewal service contracts. External systems use versioned REST and signed webhooks. Other modules must not directly query Domain Manager pricing, renewal-action or Vault-link tables as their primary integration.

## 6. Hard vs Optional
Hard: Perfex native customer/contact/project/contract/invoice/payment/email/notification/cron services. Conditional hard requirement: Vault only when legacy plaintext credentials must be retained/migrated. All other ecosystem modules are optional.

## 7. Circular Dependency Prevention
- Handoff calls associations; Domain Manager does not require Handoff.
- Onboarding stores onboarding state; Domain Manager only exposes checks/events.
- Vault does not need Domain Manager tables; association is generic/reference based.
- Property Monitor consumes target metadata; Domain Manager may display its summary via adapter.
- MCP/Ara consumes services/API; Domain Manager has no AI-runtime requirement.

## 8. Installation Behavior
At boot: verify Perfex compatibility, register core capabilities/services, detect optional integration capabilities, register adapters conditionally, never fatal because an optional module is missing. Settings should distinguish available, not installed, unconfigured and degraded states.

## 9. Shared Platform Capabilities Preferred
Reuse project-wide event/webhook runtime, API auth/scopes, idempotency/retry/dead-letter, structured audit, notifications inbox, approval orchestration, OpenBao broker and canonical identity mapping where available. Any local fallback sits behind a replaceable adapter.
