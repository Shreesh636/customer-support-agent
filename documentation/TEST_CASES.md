# Test Cases

## Executed Tests

| Test ID | Input | Expected Category | Expected Urgency | Expected Sentiment | Expected Action | Actual Result | Status |
|---|---|---|---|---|---|---|---|
| TC-01 | "How do I reset my password?" | How-to | Low | Neutral | auto_resolve | category=How-to, urgency=Low, sentiment=Neutral, action=auto_resolve, response=step-by-step password reset guidance from FAQ | ✅ PASSED |
| TC-02 | "I was charged twice for the same order and I want my money back." | Billing | High | Negative | escalate | category=Billing, urgency=High, sentiment=Negative, action=escalate, response=apology + escalation notice + request for order number/email, ticket generated | ✅ PASSED |

### TC-01 Detail — Auto-Resolve (see `screenshots/02_auto_resolve_test.png`)
- Execution status: Success in 5.28s, 1,226 tokens (per n8n execution log)
- AI Agent output:
  - `category`: How-to
  - `urgency`: Low
  - `sentiment`: Neutral
  - `action`: auto_resolve
  - `response`: A 5-step password-reset walkthrough (login page → "Forgot password?" → email → reset link → new password → confirm/login), matching the FAQ knowledge base.
  - `reason`: "User requests password reset instructions, which can be resolved with standard guidance."

### TC-02 Detail — Escalation (see `screenshots/03_escalation_test.png`)
- Execution status: Success in 5.544s, 1,393 tokens (per n8n execution log)
- AI Agent output:
  - `category`: Billing
  - `urgency`: High
  - `sentiment`: Negative
  - `action`: escalate
  - `response`: Apology for the duplicate charge, confirmation the issue is flagged for the billing team, and a request for the order number and account email to help resolve it.
  - `reason`: "The user reports a duplicate charge and requests a refund, which is a financial dispute requiring human intervention."
  - Downstream `Edit Fields` node generated a `ticket_id` (`TICKET-<timestamp>`) and `ticket_status = "Escalated"`.

---

## Recommended Additional Tests — Not Yet Executed
*(Marked explicitly as not-yet-executed; no results are claimed for these.)*

| Test ID | Input | Expected Category | Expected Urgency | Expected Sentiment | Expected Action | Actual Result | Status |
|---|---|---|---|---|---|---|---|
| TC-03 | "My account was hacked, someone logged in without my permission!" | Account | Critical | Very Negative | escalate | — | ⬜ Recommended additional test — not yet executed |
| TC-04 | "Where is my order? It's been 5 days." | Order/Delivery | Medium | Neutral/Negative | auto_resolve (ask for order number if missing) | — | ⬜ Recommended additional test — not yet executed |
| TC-05 | "Your service is a scam, I'm calling my lawyer." | Complaint | Critical | Very Negative | escalate | — | ⬜ Recommended additional test — not yet executed |
| TC-06 | "How do I update my email address on my profile?" | Account | Low | Neutral | auto_resolve | — | ⬜ Recommended additional test — not yet executed |
| TC-07 | Follow-up message referencing an earlier turn (tests memory) | (context-dependent) | (context-dependent) | (context-dependent) | (context-dependent) | — | ⬜ Recommended additional test — not yet executed |

## Testing Summary
- **2 of 2 executed tests passed**, covering both the auto-resolve and escalation branches of the workflow.
- Additional recommended tests above should be run by the student to broaden coverage (critical/security cases, delivery cases, legal-threat cases, memory continuity) before final demonstration, and this file should be updated with actual results once executed.
