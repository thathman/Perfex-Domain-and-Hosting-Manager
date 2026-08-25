# Renewal, Pricing & Billing — Acceptance Criteria

## Pricing
- [ ] Provider cost and client price are distinct.
- [ ] All agreed pricing policies calculate correctly.
- [ ] History is unchanged by later edits.
- [ ] Client cannot see provider cost/margin.
- [ ] Unpublished price blocks renewal/invoice.

## 180-Day Milestone
- [ ] Eligible client contacts receive one template email.
- [ ] Staff receives one template email.
- [ ] Re-running cron sends no duplicate.

## 60-Day Milestone
- [ ] Notices send once.
- [ ] Renewal window opens.
- [ ] Eligible client sees Renew Now.
- [ ] Unresolved pricing suppresses payable action and alerts staff.

## Early Renewal
- [ ] Renew Now creates one Perfex invoice.
- [ ] Concurrent requests still create one invoice.
- [ ] 30-day cron later skips invoice creation.

## 30-Day Milestone
- [ ] If no invoice exists, exactly one invoice is created/sent.
- [ ] Failure is visible/retryable.
- [ ] Retry does not duplicate invoice.

## Payment / Fulfilment
- [ ] Payment moves cycle to fulfilment required.
- [ ] Portal shows paid/processing, not renewed.
- [ ] Staff completion requires actual new expiry.
- [ ] Completed cycle preserves prior dates/price/cost/invoice.
- [ ] Next cycle can be derived.

## Renewal Groups
- [ ] Compatible services can share one invoice.
- [ ] Each cycle keeps distinct line reference/state.
- [ ] Incompatible grouping rejects.

## Audit / Security
- [ ] Sensitive actions are audited.
- [ ] No secrets appear in logs/events/templates.
