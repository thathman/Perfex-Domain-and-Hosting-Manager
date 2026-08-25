# Perfex Domain & Hosting Manager — Product Specification

**Spec status:** Implementation-ready  
**Spec version:** 1.0  
**Target product generation:** 2.0 architecture  
**Last updated:** 2026-08-25  
**Module slug:** `domain_manager`  
**Primary system:** Perfex CRM  

## 1. Product Vision

Perfex Domain & Hosting Manager is the authoritative business-service layer for client web infrastructure managed through Perfex. It connects domains, hosting, renewal obligations, pricing, invoices, projects, contracts, providers, client self-service, credentials, monitoring context, and future provider automation without duplicating the systems that already own those concerns.

The module is not an expiry-date spreadsheet and not a generic credential store. Its job is to answer, reliably:

- What web-infrastructure services do we manage for this customer?
- Who owns each service and which projects/contracts does it belong to?
- When does it renew?
- What does the provider charge us?
- What do we charge the client?
- Is pricing reviewed and publishable?
- Has the client been notified?
- Is the client eligible to renew now?
- Has an invoice already been created?
- Has it been paid?
- Has the provider-side renewal actually been completed?
- What changed, who changed it, and what needs attention next?

## 2. Problem Statement

The current repository implements domains and hosting as two mostly independent CRUD lists. It stores provider/expiry fields, links records to one client and one project, exposes admin profile/project tabs, and colors imminent expiries. It does not model a renewal lifecycle, billing history, client self-service, pricing policy, service bundles, provider identity, structured events, public API contracts, email templates, jobs, or secure credential ownership.

This produces operational risk:

- expiry dates can be overwritten without preserving history;
- an invoice can be forgotten or duplicated;
- provider-cost changes are not reflected safely in client pricing;
- client renewal cannot be initiated from the portal;
- credentials are currently stored and displayed as ordinary database fields;
- hosting is modeled as subordinate to one domain even though one account may serve many domains;
- client and project integration is shallow;
- there is no safe automation contract for other Perfex modules or the AI/MCP layer.

## 3. Users / Personas

### 3.1 Operations staff
Manage domains, hosting accounts, renewal dates, provider relationships, pricing, renewal completion, and exceptions.

### 3.2 Account managers
See customer infrastructure and upcoming commercial obligations without needing provider credentials.

### 3.3 Finance staff
Review renewal prices, margin, scheduled invoice creation, issued invoices, payment state, and grouped billing.

### 3.4 Technical staff
See technical relationships, linked domains/hosting, provider/control-panel metadata, projects, monitoring links, and Vault references.

### 3.5 Client contacts
See only services their organization is allowed to view, understand renewal dates/prices, and renew eligible services from the native Perfex client portal.

### 3.6 Administrators
Configure permissions, default renewal policies, pricing controls, provider definitions, email templates, integration settings, updater/release behavior, and audit access.

### 3.7 Perfex AI / MCP consumers
Read safe service state and execute permission-gated actions through shared structured service/API contracts. AI never bypasses module authorization and never receives secrets.

## 4. Goals

1. Make Perfex the business source of truth for managed domain/hosting service lifecycle.
2. Establish a normalized service model that supports domains and hosting now and SSL/DNS/server/email service types later.
3. Make renewal automation deterministic, auditable, idempotent, and billing-aware.
4. Support the agreed default lifecycle:
   - 180 days before expiry: notify staff and client.
   - 60 days before expiry: notify staff and client; open client self-renewal.
   - 30 days before expiry: automatically create and send the Perfex renewal invoice if no invoice already exists.
5. Allow a client who renews between 60 and 31 days before expiry to trigger invoice creation immediately; the later 30-day job must no-op safely.
6. Keep provider cost and client price separate, with immutable price and renewal history.
7. Provide pricing-policy and price-review controls so automated invoicing never uses unresolved pricing.
8. Replace giant create/edit forms with a guided creation workflow and a post-create service workspace.
9. Eliminate password/API-secret ownership from this module.
10. Integrate deeply but selectively with native Perfex objects and ecosystem modules.
11. Be API/event capable by design.
12. Use the hardened GitHub Releases updater direction established in Perfex Magic Login.
13. Support local-first testing; do not depend on GitHub Actions for CI.

## 5. Non-Goals

- Becoming a registrar, hosting control panel, DNS server, or payment gateway.
- Replacing Perfex invoices, payments, subscriptions, contracts, projects, customers, contacts, notifications, or email templates.
- Replacing Perfex Vault for client-facing/user credentials.
- Storing machine/service API secrets; OpenBao owns those.
- Rebuilding support/chat; Chatwoot remains the new-support communication source of truth.
- Building a separate onboarding engine; Perfex Onboarding orchestrates onboarding.
- Building a generic form builder; Perfex Forms owns structured collection.
- Hard-coding AI logic into this module.
- Automatically interpreting contract legal language.
- Treating payment as proof that the provider-side renewal completed.
- Requiring any external provider API for the core product to function.

## 6. Design Principles

1. **Perfex-native first.** The module should look and behave like first-party Perfex functionality.
2. **One owner per datum.** Reference data owned by other modules/systems; do not duplicate it.
3. **Lifecycle over CRUD.** Model state transitions and history, not just the latest row.
4. **Automation must be replay-safe.** Jobs, webhooks, and user actions are idempotent.
5. **Human-review gates where money can be wrong.** Unresolved pricing blocks invoice generation.
6. **Progressive disclosure.** Users see what they need for the current step.
7. **Client UX is a product surface, not an admin table.**
8. **Secrets by reference only.**
9. **Events before tight coupling.**
10. **Manual-first integrations.** External provider APIs enhance the system but never become a core dependency.
11. **Audit important actions.**
12. **Preserve extension points in MVP.**

## 7. Terminology

### Service
A managed renewable/non-renewable commercial or technical asset. MVP service types are Domain and Hosting.

### Service Bundle
A logical grouping representing a website/application/property context, e.g. `example.com` bundle containing domain, hosting, SSL, DNS, project, contract and monitoring relationships.

### Provider
A normalized external provider/registrar/host definition such as Cloudflare, Hostinger, Namecheap, DigitalOcean, or a generic/manual provider.

### Renewal Policy
The configured milestone/action schedule for a service or inherited default.

### Renewal Cycle
A single renewal period with immutable snapshot data: dates, client price, provider cost, invoice, payment/renewal status, and completion data.

### Renewal Window
Period during which client self-renewal is allowed. Default starts 60 days before expiry.

### Price Policy
Rule used to determine a service’s next client price. Examples: fixed, manual review, percent increase, fixed increase, cost-plus percentage, cost-plus fixed amount.

### Published Renewal Price
The price approved/finalized for the active renewal cycle and safe to expose to the client / use for invoice generation.

### Billing Group
A set of renewable services that may be consolidated onto one Perfex invoice as separate line items.

### Provider Renewal
The actual external action that extends the service at the registrar/host. Payment alone does not equal provider renewal.

## 8. Major Capabilities

### 8.1 Service Inventory
- Domain and Hosting as first-class service types.
- Service bundles.
- Provider normalization.
- Customer ownership.
- Contact visibility.
- Project, contract and other Perfex relationships.
- Active/archived/cancelled lifecycle.
- Search, filtering, saved operational views where useful.
- Bulk operations with permission controls.

### 8.2 Domain Management
- Domain name / normalized hostname.
- Registrar.
- registration/purchase date.
- expiry date.
- registration status.
- auto-renew metadata.
- DNS provider metadata.
- nameserver metadata.
- transfer/EPP status metadata without storing secret transfer codes.
- privacy/WHOIS metadata.
- linked services and bundle.
- pricing and renewal policy.

### 8.3 Hosting Management
- Provider.
- plan/package.
- account/reference name.
- start and renewal dates.
- billing cycle.
- control-panel type and safe URL metadata.
- server/hostname/IP metadata where non-secret.
- resource/package metadata where relevant.
- one hosting account linked to many domains.
- pricing and renewal policy.

### 8.4 Future Service Types
Architectural extension points for SSL, DNS management, server/VPS, business email and other infrastructure products. These are not all MVP deliverables.

### 8.5 Pricing
- Separate provider cost from client price.
- Currency and billing period.
- Current and future effective prices.
- Price policies:
  - fixed;
  - manual review;
  - percentage increase;
  - fixed increase;
  - cost-plus percentage;
  - cost-plus fixed amount;
  - one-off next-renewal override.
- Minimum price / minimum margin guardrails.
- Pricing review required state.
- Price history.
- Renewal-cycle snapshots.
- Bulk price-review/adjustment preview before commit.
- Contract-linked policy constraints represented as structured rules, not AI legal interpretation.
- Provider cost changes can trigger review.
- Provider cost never appears in client-facing merge fields or APIs.

### 8.6 Renewal Lifecycle
Default policy:
1. **180 days before expiry**
   - staff email;
   - client email;
   - in-app notification where enabled.
2. **60 days before expiry**
   - staff email;
   - client email;
   - renewal window opens;
   - client portal shows `Renew Now` when price is published and permissions allow.
3. **30 days before expiry**
   - if no renewal invoice exists, create Perfex invoice;
   - send invoice using Perfex’s normal invoice/email flow;
   - record idempotent action result.
4. Optional configurable later reminders (e.g. 14/7/1 days) may target unpaid/unresolved cycles.
5. Expiry handling records service risk/expired state but does not silently fabricate provider actions.

The system must support policy overrides per service while preserving global defaults.

### 8.7 Client Self-Renewal
From the configured window (default 60 days):
- Show renewal date and published price.
- If no invoice exists: `Renew Now` creates exactly one invoice and routes to normal Perfex invoice/payment.
- If invoice exists and is unpaid: show `View / Pay Renewal Invoice`.
- If paid but provider renewal is pending: show `Renewal Paid — Processing`.
- If completed: show new term/expiry and renewal history.
- If pricing is unresolved: show `Renewal Available Soon` and alert staff; do not expose an unapproved amount.
- Some services may use `Request Renewal` instead of immediate invoice creation when approval is required.

### 8.8 Billing & Invoice Integration
- Native Perfex invoices are the invoice source of truth.
- Service renewal line items carry durable service/renewal metadata.
- Scheduled and client-triggered invoice creation share one idempotency path.
- Payment state comes from Perfex.
- Renewal groups can consolidate several services into one invoice while preserving individual renewal-cycle accounting.
- Failed invoice creation is visible, retryable and auditable.
- Invoice deletion/cancellation must reconcile renewal state safely.
- Paid does not equal renewed.

### 8.9 Notifications & Email Templates
Use native Perfex email templates, not hard-coded bodies.

Required MVP template categories:
- 180-day client renewal notice;
- 180-day staff renewal notice;
- 60-day client renewal-window-open notice;
- 60-day staff renewal notice;
- renewal invoice created/sent;
- pricing review required (staff);
- renewal paid / provider action required (staff);
- renewal completed (client and/or staff);
- renewal automation failure (staff/admin).

Safe merge fields include:
- `{dhm_service_name}`
- `{dhm_service_type}`
- `{dhm_domain_name}`
- `{dhm_provider_name}`
- `{dhm_expiry_date}`
- `{dhm_days_until_expiry}`
- `{dhm_renewal_price}`
- `{dhm_previous_price}`
- `{dhm_price_difference}`
- `{dhm_price_change_percent}`
- `{dhm_renewal_date}`
- `{dhm_invoice_number}`
- `{dhm_invoice_link}`
- `{dhm_client_portal_link}`
- `{dhm_renewal_summary}`

Staff-only safe merge fields may include provider cost/margin. Secret values and Vault contents are prohibited from merge fields.

### 8.10 Vault Integration
- No password fields in service forms.
- No plaintext credential columns in the target model.
- Link one or more Perfex Vault entries/access packs to services/bundles.
- Vault owns reveal, access templates, requirements, audit and secret lifecycle.
- Domain & Hosting Manager stores only stable Vault references and display-safe metadata.
- Existing legacy plaintext secrets are never displayed in the new UI/API. Upgrade handling is defined in DATA_MODEL.md and DECISIONS.md.

### 8.11 Provider Integrations
Core operation is manual-provider compatible.

Future adapters may support:
- registrar expiry/status/auto-renew checks;
- hosting account metadata;
- DNS provider metadata;
- cPanel/WHM;
- Plesk;
- Cloudflare;
- Hostinger;
- other registrar/hosting APIs.

Machine/service credentials are OpenBao references. Adapters use the shared integration runtime for retries, signature verification, dead-lettering, replay, logs and observability when available.

### 8.12 Admin Dashboard
Landing page should summarize:
- active domains;
- hosting accounts;
- renewals due by window;
- invoice generation due;
- unpaid renewal invoices;
- expired/at-risk services;
- pricing reviews required;
- renewal revenue;
- provider costs/margin where permission allows;
- automation failures.

Primary navigation:
`Overview | Services | Domains | Hosting | Renewals | Pricing Reviews | Providers | Activity | Settings`

### 8.13 Creation Workflow
Replace giant forms with a guided flow:

1. **Service Type** — Domain / Hosting.
2. **Client & Relationships** — customer, contacts/visibility, project(s), service bundle, contract, assigned staff.
3. **Service Details** — type-specific fields.
4. **Pricing & Renewal** — cost, client price, policy, expiry/renewal dates, self-renewal, invoice schedule, renewal policy.
5. **Review & Create**.

Contextual entry points pre-fill relationships:
- Customer → Domains & Hosting → Add Service.
- Project → Infrastructure → Add Service.
- Bundle → Add Service.

### 8.14 Service Workspace
After creation, use a workspace, not the wizard:

`Overview | Technical | Pricing | Billing & Renewals | Renewal Schedule | Vault | Relationships | Activity`

The overview prioritizes current status, expiry, provider, customer/project context, next renewal, renewal price, invoice state and action required.

### 8.15 Client Portal
Native Perfex experience, responsive/mobile usable:
- service cards, not admin data tables;
- filter by active/renewal due;
- safe service details;
- renewal eligibility and `Renew Now`;
- invoice/payment link;
- renewal history;
- status/progress;
- contact-level visibility and renewal permissions;
- no provider cost, internal margin, internal notes, Vault secrets, provider API data or admin-only audit events.

### 8.16 Perfex Native Integrations
Meaningful native relationships:
- Customers: service ownership and customer-level tab.
- Contacts: portal permissions and notification recipients.
- Projects: infrastructure tab / service relationships.
- Contracts: pricing/term relationship and service context.
- Invoices: renewal billing source of truth.
- Payments: payment state.
- Subscriptions: optional future relationship for recurring billing where appropriate; do not force every renewal into subscriptions.
- Tasks: operational follow-up links; no separate task engine.
- Estimates/Proposals: future quoting/upsell link.
- Custom fields: supported on service records through module-specific field hooks where practical.
- Email templates: all module emails.
- Notifications: native in-app notifications.
- Activity logs: human-readable native activity plus structured audit where required.
- Cron: renewal scheduler, reconciliation and retry entry points.

## 9. Functional Requirements

### FR-001 Service identity
Every service has stable ID, type, display name, customer, lifecycle status, created/updated timestamps, and audit metadata.

### FR-002 Customer boundary
A service belongs to exactly one Perfex customer. Cross-customer relationships are forbidden.

### FR-003 Many-domain hosting
A hosting account can relate to zero-to-many domain services.

### FR-004 Service bundles
Services can be grouped under a bundle; bundles are customer-scoped.

### FR-005 Provider normalization
Provider references use provider records rather than repeated free text; a generic/manual provider path is supported.

### FR-006 Renewal policy inheritance
A service inherits the global renewal policy unless an explicit service override exists.

### FR-007 Default milestones
The shipped default policy has 180-day notice, 60-day renewal opening, and 30-day invoice generation/sending.

### FR-008 Pricing publication gate
Client self-renewal and automatic invoice generation require a published renewal price.

### FR-009 Renewal-cycle uniqueness
Only one active renewal cycle may represent a given service and renewal term.

### FR-010 Invoice uniqueness
Only one primary renewal invoice line/reference may be generated for the same service renewal cycle unless an authorized correction/credit workflow creates a replacement relationship.

### FR-011 Early client renewal
Client-triggered renewal inside the open window can create the invoice before the 30-day scheduler.

### FR-012 Cron idempotency
Repeated scheduler runs do not duplicate emails, invoices, events or renewal cycles.

### FR-013 Payment reconciliation
Paid Perfex invoices advance the renewal cycle to a `paid / renewal required` state; they do not mark the service renewed.

### FR-014 Manual renewal completion
MVP allows authorized staff to mark provider-side renewal complete, recording new expiry date and actor.

### FR-015 Renewal history
Completed/failed/cancelled cycles remain queryable and are not overwritten by the next cycle.

### FR-016 Price history
Price changes are effective-dated and historical values remain immutable.

### FR-017 Cost confidentiality
Provider cost/margin is restricted by staff permission and excluded from client output.

### FR-018 No secret storage
New schema and APIs contain no password/API-secret fields.

### FR-019 Vault references
Services/bundles can link to Vault resources without reading secret values.

### FR-020 Audit
Pricing, invoice generation, policy changes, renewal actions, permission-sensitive reads, bulk changes and integration failures are auditable.

### FR-021 API-first service layer
UI controllers call service/application-layer methods that are also reusable by API/MCP adapters.

### FR-022 Events
Meaningful state changes emit namespaced events after successful transaction commit.

### FR-023 External integration safety
Webhooks/jobs use correlation IDs, idempotency keys, retries/backoff and dead-letter/replay through shared integration facilities where available.

### FR-024 Update safety
The module supports signed/validated release artifact flow per updater architecture.

## 10. Non-Functional Requirements

### Security
- Server-side authorization on every read/write/action.
- Native CSRF protection for browser writes.
- Output escaping / allowlisted rich text.
- Parameterized/query-builder database access.
- No secret values in logs/events/templates/API.
- Signed webhook validation.
- Replay protection for webhooks.
- Least-privilege OpenBao/Vault references.
- Sensitive financial data permission separation.

### Reliability
- Scheduler actions idempotent.
- Database changes transactional where cross-record consistency matters.
- Retryable external/invoice failures.
- Dead-letter visibility.
- No silent action loss.

### Performance
- Indexed renewal-date/status/customer/provider queries.
- Paginated admin/API lists.
- Dashboard uses aggregated/index-friendly queries.
- Client portal scopes before query execution.

### Maintainability
- Controllers thin.
- Domain/service, pricing, renewal, invoice orchestration and provider adapters separated.
- No direct cross-module table writes.
- Stable interfaces/events.
- Contiguous migrations.

### Compatibility
- Keep module slug `domain_manager` unless a future ADR explicitly changes it.
- Current repository is treated as an imported 1.0.1 prototype with unknown install footprint; do not assume greenfield.
- Upgrade path must preserve non-secret business data and safely handle legacy plaintext credentials.

## 11. Admin Experience

### Dashboard
Operational overview plus actionable queues:
- Renewals opening soon.
- Invoices due to generate.
- Pricing reviews blocking automation.
- Paid but not renewed.
- Expired/unresolved.
- Automation failures.

### Customer profile
`Domains & Hosting` tab:
- bundle/service summary;
- upcoming renewals;
- invoices/payment state;
- add service contextual action;
- related projects/contracts.

### Project
`Infrastructure` tab:
- bundles/services tied to project;
- production/staging domain roles;
- hosting and renewal state;
- Vault links shown as references/actions only.

### Pricing review
Queue with filters and bulk preview:
- old provider cost/client price;
- proposed cost/client price;
- delta and margin;
- policy/reason;
- accept/override/hold.
No bulk price write occurs without preview and confirmation.

### Provider management
Provider records, capability flags and non-secret metadata. Machine-secret configuration stores only OpenBao reference identifiers.

## 12. Client Experience

Client-facing lifecycle states:
- Active.
- Renewal available.
- Invoice issued / payment due.
- Paid — processing.
- Renewed.
- Expired/attention required.
- Renewal unavailable because pricing/approval pending (friendly copy).

Portal must not reveal internal terminology like dead-letter queues, provider margins, audit correlation IDs, or internal exception messages.

## 13. Reporting

MVP:
- services by customer/type/status;
- renewals due by date range;
- renewal invoices created/paid/unpaid;
- paid but not completed;
- pricing review required;
- annualized renewal revenue and provider cost/margin for authorized staff;
- provider concentration.

Later:
- renewal conversion timing;
- churn/non-renewal reasons;
- margin trends;
- provider price-change impact;
- automation reliability.

## 14. Observability

Structured logs/metrics should include:
- job execution count/duration;
- renewal steps evaluated/executed/skipped;
- invoice creation success/failure;
- notification send result;
- integration retry/dead-letter counts;
- webhook signature/idempotency rejection counts;
- renewal cycles stuck by state/age.

Every scheduler/integration operation carries a correlation ID.

## 15. Edge Cases

- Expiry date is missing or invalid.
- Service already expired when imported.
- Client removed/inactivated.
- Contact loses renewal permission while invoice exists.
- Service changes customer (normally forbidden; use explicit controlled transfer).
- Hosting account serves domains owned by different customers (forbidden in MVP; split or explicit managed-shared-host exception later).
- Invoice deleted/cancelled after generation.
- Invoice paid after expiry.
- Client clicks Renew Now concurrently in two browser tabs.
- Cron runs while client clicks Renew Now.
- Renewal price changes after invoice creation.
- Currency changes between cycles.
- Provider cost unknown.
- Contract price cap conflicts with pricing policy.
- Service is auto-renewing at provider but client invoice unpaid.
- Provider renewal fails after payment.
- Legacy record contains plaintext credentials.
- Vault/OpenBao integration unavailable.
- External webhook is duplicated/replayed.
- Renewal grouping contains services with incompatible customer/currency/tax configuration.

## 16. Acceptance Criteria — Product Level

The target architecture is acceptable when:

1. Staff can create Domain and Hosting services through the guided workflow.
2. One hosting service can link to multiple domains.
3. Service/bundle/customer/project relationships are visible contextually.
4. No new password/API-secret fields exist.
5. Vault links work without exposing secret values.
6. Pricing separates provider cost and client price with history.
7. A renewal cycle snapshots price/cost/dates.
8. The default 180/60/30 policy is seeded and functional.
9. Client self-renewal opens at 60 days by default.
10. Client-triggered renewal creates at most one Perfex invoice.
11. The 30-day cron creates/sends an invoice only when none exists.
12. Unresolved pricing blocks both client renewal and automated invoicing.
13. Payment transitions renewal to paid/processing, not renewed.
14. Authorized staff can complete renewal and set/verify new expiry.
15. Renewal and pricing history remain immutable.
16. Client portal is purpose-built and contact-permission aware.
17. API and event contracts exist for core resources/actions.
18. Audit records cover financial/renewal/policy/security-sensitive actions.
19. Module updater follows safe GitHub Release/checksum/archive/migration rules.
20. Local test suite covers idempotency, permissions, portal, migrations and updater paths.

## 17. Full Product Direction

Nothing agreed is dropped if deferred. The roadmap includes:
- SSL service type and live certificate checks.
- DNS/server/business-email service types.
- provider adapters.
- automated provider renewal where safe.
- Property Monitor / Uptime Kuma health context.
- Approval Workflow for price/exception approvals.
- Forms-based service requests.
- Onboarding orchestration hooks.
- Handoff-driven infrastructure association.
- Chatwoot contextual support actions.
- MCP/Ara tools.
- bulk pricing.
- richer renewal groups.
- estimates/proposals for new/upgrade services.
- contract-aware commercial controls.
- analytics and churn/margin reporting.
