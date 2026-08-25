# Service Catalog & Workspace — Acceptance Criteria

- [ ] Domain and Hosting can be created through the five-step workflow.
- [ ] Type-specific fields never leak into the wrong service type.
- [ ] No password/API-secret fields appear anywhere.
- [ ] Customer/project contextual creation prefills known relationships.
- [ ] Server-side validation does not trust wizard/client state.
- [ ] Hosting can relate to multiple domain services.
- [ ] Cross-customer relationships are rejected.
- [ ] Provider is normalized, not free text per record.
- [ ] Existing service opens a workspace, not the creation wizard.
- [ ] Overview shows operational/commercial next action.
- [ ] Pricing/cost visibility follows permissions.
- [ ] Vault tab contains safe references only.
- [ ] Customer tab is contextual, not a raw legacy DataTable.
- [ ] Project Infrastructure tab uses the same service data.
- [ ] Search/filter uses validated repository filters.
- [ ] Archive is POST+CSRF, not destructive GET.
- [ ] Mobile layouts remain usable.
- [ ] Create/edit/archive/relationship actions produce audit/events.
