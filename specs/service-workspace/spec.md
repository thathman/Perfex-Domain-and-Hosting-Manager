# Feature Spec — Service Catalog, Creation Flow & Workspace

## Status
Committed core capability. Required for MVP.

## Objective
Replace the current giant domain/hosting forms and two-list mental model with a coherent service catalog embedded naturally in Perfex customer/project context.

## MVP Service Types
**Domain:** domain name, registrar, registration/expiry dates, registration status, DNS provider, nameservers, privacy status, safe transfer status and external auto-renew metadata.

**Hosting:** provider, plan/package, safe account reference, start/renewal date, control-panel type/safe URL, server hostname/IP, resource/package metadata and environment.

No username/password/API-key fields.

## Creation Workflow
1. **Service Type** — Domain or Hosting.
2. **Client & Relationships** — customer required; contact defaults; bundle; projects; contract; assigned staff. Contextual entry points prefill known values.
3. **Service Details** — type-specific only.
4. **Pricing & Renewal** — provider cost, client price, currency, billing cycle, price policy, expiry/renewal date, self-renew, renewal policy/default, billing group.
5. **Review** — readable summary; server validates all cross-record rules.

## Post-Create Workspace
Tabs: Overview, Technical, Pricing, Billing & Renewals, Renewal Schedule, Vault, Relationships, Activity.

Overview prioritizes service/customer/provider/status, expiry/days remaining, renewal state, published price, invoice/payment state, action required and bundle/project/contract context. Technical is safe type metadata. Pricing shows cost/margin only with permission. Billing/Renewals shows cycles/invoices/fulfilment/history. Renewal Schedule shows inherited/default + overrides. Vault shows safe references/actions only. Relationships shows bundle/domain↔hosting/Perfex links. Activity combines human-readable activity + permitted structured audit summary.

## Dashboard / Lists
Operational cards/queues: active domains, hosting accounts, renewal windows open, invoices due, unpaid renewal invoices, pricing reviews, paid-renewal-required, expired/at-risk, automation failures and authorized renewal revenue/cost/margin.

Filters: search, type, customer, provider, status, project, bundle, renewal date/state and pricing review. All filtering is validated/query-builder based.

## Native Context
Customer `Domains & Hosting`: active service summary, upcoming renewals, invoice/payment state, Add Service and bundle/project links.

Project `Infrastructure`: linked services/bundle, domain roles, hosting, renewal risk and Add/Attach Service.

## Responsive / Empty States
Wizard/workspace/actions remain mobile-usable. Dense tables have card/overflow strategy. Empty states are actionable and never suggest local password storage when Vault is absent.

## Validation
Customer required; valid normalized domain; same-customer bundle/service/project/contract relations; hosting may relate to many domains of same customer; safe URLs/money/dates; generic metadata rejects secret-like fields.

## Events
`domain_hosting.service.created/updated/archived`, bundle created/updated and relationship created/removed.

## Out of Scope
Provider control-panel management, secret reveal, duplicate monitoring, onboarding engine and support/chat UI.
