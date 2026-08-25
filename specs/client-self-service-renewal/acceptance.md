# Client Self-Service Renewal — Acceptance Criteria

- [ ] Portal is native Perfex and customer/contact scoped.
- [ ] Default renewal window opens at 60 days.
- [ ] Before 60 days, Renew Now is unavailable.
- [ ] At 60 days with published price, eligible contact sees Renew Now.
- [ ] Contact without renewal permission cannot execute action.
- [ ] Renew Now creates at most one invoice.
- [ ] Existing invoice changes action to View / Pay.
- [ ] Payment shows Paid — Processing, not Renewed.
- [ ] Completed renewal shows new expiry/history.
- [ ] Pricing unresolved shows no client price/action and alerts staff.
- [ ] Two-tab/double-click race produces one invoice.
- [ ] Cron/client race produces one invoice.
- [ ] Client cannot alter invoice amount by request tampering.
- [ ] Client cannot access another customer's service by ID.
- [ ] Provider cost/margin/internal notes/secrets are absent from HTML/API.
- [ ] Mobile renewal action is usable.
- [ ] Audit/event records identify contact actor and correlation ID.
