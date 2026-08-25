# Perfex Domain & Hosting Manager

A Perfex CRM module for managing client domains and hosting services.

## Current Status

**Planning / architecture rewrite in progress.**

The code currently on `main` is an imported prototype (declared module version 1.0.1). It currently provides:
- admin domain records;
- admin hosting records;
- basic client/project links;
- expiry-date display/status coloring;
- basic Perfex staff permissions;
- customer-profile and project admin tabs.

The current implementation should **not** yet be considered the target production architecture.

Known limitations include flat CRUD, no renewal scheduler, no invoice/payment automation, no true client self-service portal, no price/renewal history, no public REST API/events/webhooks, no hardened updater, legacy plaintext username/password fields and legacy third-party purchase-verification code.

## Planned Product

The approved product direction turns this into a native Perfex managed-infrastructure lifecycle system with:
- Domain and Hosting service types;
- service bundles;
- normalized providers;
- modern guided creation/workspace UX;
- provider cost vs client price;
- price adjustment policies and history;
- renewal cycles;
- default 180/60/30 renewal schedule;
- client self-renewal from 60 days before expiry;
- automatic Perfex renewal invoice creation at 30 days if the client has not already renewed;
- native Perfex invoice/payment integration;
- renewal groups;
- client portal;
- email templates/merge fields;
- Vault credential references;
- API/events/audit;
- safe GitHub Releases updater architecture.

These capabilities are **planned** until implemented and validated.

## Future Direction
Later phases include SSL/certificate management, provider/registrar adapters, Cloudflare/Hostinger/cPanel/Plesk integrations, Property Monitor/Uptime Kuma context, Handoff/Onboarding/Forms/Approval Workflow integrations, Chatwoot contextual support, MCP/Ara tools and provider-side renewal automation where safe.

## Architecture Boundaries
- **Perfex CRM** — customers, contacts, projects, contracts, invoices, payments, email templates, notifications and client identity.
- **Domain & Hosting Manager** — managed service inventory, pricing, renewal policy/cycles and renewal orchestration.
- **Perfex Vault** — human/client credentials.
- **OpenBao** — machine/service/provider API secrets.
- **Chatwoot** — new support conversations.
- **Perfex Onboarding** — onboarding orchestration.
- **Perfex Forms** — structured form collection.
- **Property Monitor / Uptime Kuma** — operational monitoring data.
- **MCP / Ara** — AI orchestration over permission-aware service contracts.

No new plaintext credentials should be introduced into this module.

## Product Documentation
Start with [`docs/product/README.md`](docs/product/README.md), then:
- [`docs/product/PRODUCT_SPEC.md`](docs/product/PRODUCT_SPEC.md)
- [`docs/product/MVP.md`](docs/product/MVP.md)
- [`docs/product/BUILD_PHASES.md`](docs/product/BUILD_PHASES.md)
- [`docs/product/IMPLEMENTATION_TASKS.md`](docs/product/IMPLEMENTATION_TASKS.md)
- [`docs/product/ARCHITECTURE.md`](docs/product/ARCHITECTURE.md)
- [`docs/product/DATA_MODEL.md`](docs/product/DATA_MODEL.md)
- [`docs/product/API.md`](docs/product/API.md)
- [`docs/product/INTEGRATIONS.md`](docs/product/INTEGRATIONS.md)
- [`docs/product/PERMISSIONS.md`](docs/product/PERMISSIONS.md)
- [`docs/product/TEST_PLAN.md`](docs/product/TEST_PLAN.md)
- [`docs/product/DECISIONS.md`](docs/product/DECISIONS.md)
- [`docs/product/GAP_ANALYSIS.md`](docs/product/GAP_ANALYSIS.md)

Feature specs are under [`specs/`](specs/).

## Perfex Requirement
The current prototype metadata declares Perfex `3.0.*` compatibility. Phase 0 must verify the actual supported Perfex/PHP baseline rather than assuming the historical declaration is sufficient.

## Installation — Current Prototype
For the current code only:
1. Copy `domain_manager` into the Perfex `modules/` directory.
2. Activate **Domain Manager**.
3. Review staff permissions before use.

**Security note:** the prototype contains legacy credential fields. Do not use them for new secrets. The target architecture removes them and uses Perfex Vault references.

## Development Workflow
Project direction is local-first:
- development/tests run locally;
- Dell-hosted Perfex staging is used for integration validation;
- GitHub Actions is not required for CI;
- GitHub Releases host validated release artifacts;
- published tags/releases are immutable.

When the project-wide Forgejo/local-git workflow is active, development should be committed/tested there before public release promotion.

## Release Direction
Target release flow: semantic version → GitHub Release → module ZIP + SHA-256 → archive/version validation → backup → contiguous migrations → staging validation → production. Never rewrite or retarget a published release tag.

## License / Ownership
Project ownership/licensing terms should be documented before the first production release of the rewritten module. The legacy Hopperstack/Envato purchase-verification mechanism is not part of the approved target architecture.
