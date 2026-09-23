# Analysis Questions

### 1. Why should a customer support agent classify a request before generating the final response?
Classification tells the system *what kind of problem it's dealing with* before it decides how to respond. Without classifying category, urgency, and sentiment first, the agent has no reliable basis for choosing between auto-resolving and escalating, and it risks giving a generic or unsafe answer to a request that actually needed human review (e.g., treating a refund request like a simple how-to question). Classifying first also produces the structured data (category, urgency, sentiment) needed for ticket routing and reporting.

### 2. Which types of support requests should always be escalated to a human, and why?
Based on the escalation rules implemented in this workflow: refund/cancellation requests involving money, financial disputes (duplicate/incorrect charges, payment problems), account security issues (hacking/unauthorized access), legal concerns, serious complaints, and very negative customer sentiment. These are escalated because they involve financial liability, legal exposure, account safety, or a level of customer distress that requires human judgment, empathy, and authority the AI does not have — the AI must never invent refund amounts, policies, or promises on the company's behalf.

### 3. How does conversation memory improve the quality of customer support?
Conversation memory (the `Simple Memory` node, keyed to the session ID) lets the agent recall earlier messages in the same conversation instead of treating every message in isolation. This means the customer doesn't have to repeat information already given, the agent's follow-up questions and responses stay consistent with what was discussed earlier, and multi-turn issues (e.g., "yes, my order number is 12345" after being asked for it) can be resolved coherently.

### 4. How can urgency, sentiment, customer tier, and issue category influence ticket routing?
- **Urgency** (Low–Critical) determines how quickly a ticket should be handled once escalated.
- **Sentiment** (Positive–Very Negative) flags customers who need more careful, empathetic handling or faster response, and very negative sentiment alone can trigger escalation.
- **Issue category** (Billing, Account, Complaint, etc.) determines which team or queue the ticket should be routed to.
- **Customer tier** (not currently implemented in this workflow, but a natural extension) could raise priority or route VIP customers to specialized support staff regardless of urgency/sentiment.
Together these signals let a routing system prioritize and direct tickets automatically instead of a human manually reading every message first.

### 5. What risks could occur if a support agent is given full automation without escalation rules?
Without escalation rules, the agent could auto-approve or promise refunds/cancellations it has no authority or accurate information to grant, invent company policies or prices, mishandle account-security incidents (e.g., not detecting an active account-takeover situation), fail to recognize legal threats needing careful human handling, and give inconsistent or reputation-damaging responses to very upset customers. This creates real financial, legal, and trust risks for the business — which is exactly why this workflow enforces mandatory escalation for high-risk categories rather than trusting the AI to self-limit only through instructions.
