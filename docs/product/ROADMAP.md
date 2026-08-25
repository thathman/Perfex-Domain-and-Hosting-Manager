# Perfex Domain & Hosting Manager — Roadmap

## Status Legend
- **Committed** — agreed product direction.
- **Planned** — sequenced after prerequisites.
- **Exploratory** — valid future opportunity; not committed to a specific release.
- **Rejected** — intentionally not part of the product.

## MVP — Committed
- New normalized service architecture.
- Domain and Hosting service types.
- Service bundles.
- Provider records.
- Customer/project/contract relationships.
- Modern guided create workflow.
- Service workspace.
- Separate provider cost/client price.
- Pricing policies and history.
- Pricing publish/review gate.
- Renewal cycles and history.
- Default 180/60/30 renewal schedule.
- Native Perfex email templates/notifications.
- 60-day client self-renewal window.
- Early `Renew Now` invoice creation.
- 30-day automatic invoice creation/send.
- Native Perfex invoice/payment integration.
- Renewal groups foundation.
- Manual provider-side renewal completion.
- Client portal.
- Customer/project contextual views.
- Vault references; no plaintext credentials.
- API/event-ready application services.
- Structured audit.
- Safe updater/release architecture.
- Local test/staging workflow; no GitHub Actions dependency.

## v1.x — Planned

### Operational quality
- Saved admin views.
- richer bulk actions.
- richer renewal-group controls.
- optional 14/7/1 unpaid reminder policy presets.
- price-change client notice templates/policy.
- contract-linked pricing guardrails.
- estimates/proposals for service upgrades/new products.
- subscription relationship where a use case genuinely fits.

### Service coverage
- SSL service type.
- certificate expiry/status checks.
- DNS-management service type.
- server/VPS service type.
- business-email service type where useful.

### Ecosystem integrations
- Handoff associations.
- Onboarding actions/completion checks.
- Forms-driven service requests.
- Notifications Inbox routing.
- Approval Workflow for price exceptions and bulk price changes.
- Organisation Self-Service client admin controls.
- Property Monitor/Uptime Kuma health context.
- Chatwoot contextual service details and renewal support actions.

## v2 / Advanced Vision — Planned/Exploratory

### Provider automation
- Provider adapter SDK/interface.
- registrar status/expiry sync.
- Cloudflare adapter.
- Hostinger adapter.
- cPanel/WHM adapter.
- Plesk adapter.
- provider cost synchronization.
- automatic renewal initiation after payment when provider supports safe idempotent renewal and policy allows it.
- domain transfer orchestration.

### Commercial intelligence
- bulk provider-price-impact analysis.
- margin trends.
- renewal conversion timing.
- non-renewal/churn reasons.
- provider concentration/risk.
- forecasted renewal cash flow.

### AI / MCP
- Ara/MCP read tools for customer services, upcoming renewals, paid-but-not-renewed items and pricing-review queues.
- permission-gated actions for pricing proposals, renewal quotes, safe retries and approval requests.
- AI never reads Vault/OpenBao secret values through this module.
- human handoff routes through Chatwoot where conversational support is needed.

### External ecosystem
- WordPress/WooCommerce property/service context.
- OJS-hosting/domain relationships.
- richer property inventory integration.
- external service-account mapping.

## Future Opportunities — Exploratory
- provider benchmark pricing;
- portfolio/domain acquisition/disposition workflows;
- client renewal choices across multi-year terms;
- automatic tax/currency policy selection;
- automated safe DNS drift checks;
- cost allocation across shared infrastructure;
- shared-hosting multi-customer model with strict data separation;
- change-request approval before technical provider actions.

## Explicitly Rejected / Superseded
- **Flat password fields in Domain/Hosting records** — superseded by Vault/OpenBao ownership.
- **One giant create/edit form** — superseded by guided creation + workspace.
- **One hosting account = one domain** — superseded by service relationships.
- **Expiry coloring as the renewal system** — retained only as a visual hint; lifecycle engine owns renewal behavior.
- **Hard-coded email bodies** — native Perfex email templates own content.
- **Payment automatically means renewed** — rejected; provider fulfilment is separate.
- **Provider API required for core use** — rejected; manual provider mode remains first-class.
- **Duplicate onboarding engine** — Onboarding owns orchestration.
- **Duplicate support engine** — Chatwoot owns new support communication.
- **Duplicate form builder** — Forms owns structured collection.
- **Module-specific AI bypass** — shared MCP/Ara owns AI interaction.
- **Plaintext machine secrets** — OpenBao owns machine/service secrets.
- **Rewriting published release tags** — forbidden.
- **GitHub Actions as required CI** — rejected; tests run locally/staging.
