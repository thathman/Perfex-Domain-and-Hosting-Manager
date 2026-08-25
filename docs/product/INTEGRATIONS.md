# Perfex Domain & Hosting Manager — Integrations

## 1. Principles
- Perfex remains the business source of truth.
- Domain & Hosting Manager owns managed-service inventory, commercial renewal state and renewal orchestration.
- Cross-module integrations use service contracts/events rather than direct table access.
- Optional modules must not become circular hard dependencies.
- External side effects are idempotent and auditable.
- Secrets remain in Vault/OpenBao according to ownership.

## 2. Dependency Matrix

| System / Module | Dependency | Owns | Domain Manager consumes | Domain Manager provides |
|---|---|---|---|---|
| Perfex Customers | Required | customer/org | identity/status | service/renewal context |
| Perfex Contacts | Required | contact/portal auth | recipients/permissions | portal service/renewal UX |
| Perfex Projects | Native relationship | project | project context | Infrastructure tab/service links |
| Perfex Contracts | Native relationship | contract | contract/term context | service/pricing relationship |
| Perfex Invoices | Required for billable renewal | invoices | create/read invoice state | renewal line metadata/reference |
| Perfex Payments | Required for paid state | payments | payment state/events | fulfilment state |
| Perfex Email Templates | Required | template content/config | rendering/send | merge fields/template registrations |
| Perfex Notifications | Required | delivery | native delivery | renewal events |
| Perfex Cron | Required | scheduler execution | cron callback | scheduler/reconciliation jobs |
| Perfex Vault | Optional normally | human/client secrets | safe refs/deep links | service/bundle associations |
| OpenBao | Optional until provider automation | machine secrets | brokered secret execution | provider integration context |
| Handoff | Optional | handoff workflow | handoff events | infrastructure actions/context |
| Onboarding | Optional | orchestration | onboarding requests | actions/checks/events/deep links |
| Forms | Optional | forms/submissions | mapped request payloads | safe service actions |
| Property Monitor | Optional | operational observations | health/status | authoritative service/property links |
| Uptime Kuma | Optional | raw monitoring | monitor state | safe target metadata |
| Chatwoot | Optional | support conversations | conversation mapping | service/renewal context/handoff |
| Notifications Inbox | Optional | consolidated inbox | event subscription | renewal/failure notifications |
| Approval Workflow | Optional | approvals | approval results | price/exception requests |
| Organisation Self-Service | Optional | delegated org controls | role/capability mapping | renewal capabilities |
| Magic Login | Optional | secure login links | authenticated deep links | renewal destination metadata |
| Verified Receipts | Optional | verified receipt context | payment signals | invoice/renewal refs |
| Bachs/payment integration | Optional | payment rail | Perfex payment state | invoice context |
| Documenso | Optional indirect | documents/signatures | signed contract status | service/contract context |
| MCP / Ara | Optional | AI/tool orchestration | structured resources/actions | permission-aware contracts |
| WordPress/WooCommerce/OJS | Optional | application data | mapped property/app metadata | domain/hosting context |

## 3. Customers & Contacts
**Owner:** Perfex. **Mode:** synchronous identity/authorization reads; async notifications. Store stable IDs only. Client queries scope by authenticated customer before retrieval. Inactive/deleted customers block new renewal side effects but do not erase history. UI: customer `Domains & Hosting` tab and client portal. Audit contact visibility/notification-target changes.

## 4. Projects
**Owner:** Perfex project; Domain Manager owns association. **Contract:** attach/query service by project and role. **Mode:** synchronous/contextual. **Events:** relationship created/removed. **Permissions:** actor must see project and service. Deleted/archived project never deletes service. UI: project `Infrastructure` tab and contextual Add Service.

## 5. Contracts
**Owner:** Perfex contract content/lifecycle; Domain Manager owns structured service/pricing configuration. Reference contract ID and optional structured commercial rules such as adjustment ceiling/effective date. Do not automatically interpret contract prose. Contract status events may trigger reviews. Missing contract leaves a stale relationship rather than deleting service.

## 6. Invoices & Payments
**Owner:** Perfex. **Mode:** synchronous invoice creation; native payment/invoice hooks plus reconciliation job. `BillingOrchestrator::ensureInvoice(...)` accepts customer/currency/tax/line items/due date/renewal refs and returns created/existing invoice ID. Domain Manager emits invoice-created/payment-received/fulfilment-required events. Failures persist as retryable actions; no fabricated refs and no duplicate on retry. UI cross-links renewal, portal and invoice.

## 7. Vault
**Owner:** Vault human/client credentials. Domain Manager may link, validate and show safe metadata/deep links. There is intentionally no secret-reveal method in its integration contract. Vault enforces its own Access Templates/Requirements/Entries/Requests/Access Packs and audit. If absent, secret-related actions degrade; no local password fallback.

## 8. OpenBao
**Owner:** machine/service/provider API secrets. Provider adapters store only opaque reference/alias; secret execution happens through shared broker/runtime. Integration-config permission is separate from ordinary service edit. Failure disables automation and preserves manual mode. Audit refs/results only, never values.

## 9. Handoff
Handoff owns project completion. Domain Manager can create/attach a service bundle from project context and expose infrastructure/renewal summary. Prefer async handoff events plus synchronous attach/create action. Handoff actor must also have Domain Manager rights. Handoff decides whether missing infrastructure is blocking. Carry correlation ID into audit.

## 10. Onboarding
Onboarding owns orchestration/state. Domain Manager exposes checks/actions such as `has_active_domain_service`, `has_hosting_service`, `service_has_published_renewal_policy`, project-infrastructure linking and deep links. Domain events can satisfy onboarding steps. No independent onboarding state is created here.

## 11. Forms
Forms owns forms/submissions. Mapped submissions may request domain registration/transfer, hosting upgrade, safe service metadata or renewal/quote. Passwords/tokens/EPP codes are prohibited ordinary form fields. Submission actions use validated customer/contact context and normal Domain Manager events.

## 12. Property Monitor & Uptime Kuma
Domain Manager owns authoritative commercial/service asset and renewal data. Property Monitor owns operational property health observations; Uptime Kuma owns raw check state. Expose safe domain/endpoint/customer/project/service IDs for association. Health failures never alter renewal lifecycle. UI may show contextual health/deep link rather than duplicate monitoring.

## 13. Chatwoot
Chatwoot remains new-support conversation source of truth. Domain Manager provides safe customer service/renewal/invoice context and deep links; Chatwoot supplies conversation/contact mapping and human handoff. Messaging automation is optional. Chatwoot failure never affects renewal billing. Future UI may expose Open Support Conversation/context panel.

## 14. Notifications Inbox
Consumes renewal/pricing/failure events for consolidated display. Native Perfex email/in-app notification remains MVP delivery path.

## 15. Approval Workflow
Approval Workflow owns approval state. Domain Manager can request approval for price increases over threshold, below-margin exceptions, bulk pricing, invoice/payment overrides or future provider auto-renew enablement, and consume the result. Failure leaves the action blocked/pending; never silently bypass.

## 16. Organisation Self-Service
Provides organization-level delegated role mapping (e.g. org admin, billing contact, technical contact). Domain Manager consumes mapping but performs final authorization. If absent, native/contact-level permissions apply.

## 17. Magic Login
Magic Login owns one-time/authenticated login links. Domain Manager may request a secure portal deep link for renewal communication but never creates its own login tokens. If unavailable, use normal Perfex portal URL.

## 18. MCP / Ara
MCP/Ara owns AI orchestration/permissions/human handoff. It consumes Domain Manager services/API to read inventory, upcoming renewals, pricing-review and paid-but-not-renewed queues; safe actions may create drafts/proposals/retries/approval requests or complete renewal only with explicit permission. No DB bypass and no secrets. Conversational human handoff uses Chatwoot.

## 19. WordPress / WooCommerce / OJS / Property Extensions
These systems keep their own content/order/journal/property data. Domain Manager exposes safe domain/hosting context and stable service IDs. Durable mapping is preferred over email-address-only matching.

## 20. Other Ecosystem Components
- Verified Receipts/Bachs consume/produce Perfex payment context; Domain Manager follows Perfex invoice/payment state rather than integrating payment rails directly.
- Documenso influences contract/document status through its owning integration; Domain Manager stores contract refs only.
- Appointment has no direct dependency by default.

## 21. Install & Failure Rules
- Perfex core is the only hard runtime platform dependency.
- Optional integrations register by capability detection and boot safely when absent.
- No direct cross-module table writes or primary `require` of another module's models.
- Legacy-secret migration is the one case where Vault availability or an explicit authorized discard decision is required before secret cleanup.
- Integration failures are observable, retryable where safe, audited and never silently mutate unrelated business state.
