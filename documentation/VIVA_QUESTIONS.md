# Viva Questions & Answers

**1. What is n8n?**
n8n is a workflow automation platform that lets you connect services and logic visually using nodes, including native support for building AI agent workflows (LangChain-based nodes).

**2. What is an AI Agent in this context?**
It's a specialized n8n node (`@n8n/n8n-nodes-langchain.agent`) that sends a prompt (system message + user input) to an LLM, optionally uses memory and tools, and can enforce a structured output format — acting as the "brain" of the workflow.

**3. Why was Groq chosen as the LLM provider?**
Groq provides fast, low-latency inference for open-weight models like `openai/gpt-oss-120b`, which suits an interactive chat-based support workflow where quick responses matter.

**4. Why use a Structured Output Parser instead of parsing plain text?**
It forces the LLM to return a fixed JSON schema every time, so downstream nodes (`If`, `Edit Fields`) can reliably reference exact field names instead of relying on fragile text parsing or regex on free-form responses.

**5. Why use an IF node here?**
The `If` node makes the auto-resolve vs. escalate decision explicit and deterministic in the workflow, based on the `action` field the AI Agent returned, cleanly separating "safe automation" from "needs a human" paths.

**6. Why use memory in this workflow?**
`Simple Memory` lets the agent remember the last 10 messages in a session, so it can handle multi-turn conversations (e.g., asking for an order number and remembering the answer) instead of treating each message independently.

**7. What is a session ID and why does it matter?**
The session ID (provided by the Chat Trigger) uniquely identifies a conversation so the memory node can store and retrieve the correct conversation history per customer, rather than mixing up different customers' conversations.

**8. What is the FAQ/knowledge base in this project?**
It's a block of general support guidance (password reset, order tracking, profile updates, login problems, billing/refund guidance) embedded directly in the AI Agent's system prompt, used to answer common low-risk questions consistently.

**9. Why must financial disputes always be escalated?**
Because they involve real money and legal/financial risk to the company; an AI should not have unilateral authority to approve refunds, confirm charges are erroneous, or make binding financial decisions.

**10. What's the difference between auto-resolve and escalation in this workflow?**
Auto-resolve means the AI Agent answers directly using safe, general guidance (`ticket_status = "Auto-Resolved"`). Escalation means the AI flags the case for a human, generates a ticket ID, and clearly tells the customer it's being forwarded to a human (`ticket_status = "Escalated"`).

**11. How is sentiment detection used here?**
The AI Agent classifies each message's sentiment (Positive/Neutral/Negative/Very Negative) as part of its structured output; "Very Negative" sentiment alone is one of the explicit triggers for escalation, regardless of category.

**12. How is urgency detection used?**
Urgency (Low/Medium/High/Critical) is classified alongside category and sentiment, giving a ticket a priority signal that (in a full production system) would drive how quickly a human team responds.

**13. What is prompt engineering, and how is it applied here?**
Prompt engineering is the practice of carefully designing an LLM's instructions to produce reliable, desired behavior. Here it's applied through clearly separated sections (role, taxonomies, escalation rules, safety rules, FAQ, output schema) so the model behaves predictably across different customer inputs.

**14. How does this workflow prevent hallucination?**
The system prompt explicitly instructs the AI to never invent company policies, prices, refund rules, delivery promises, or account information, and to ask a follow-up question instead of guessing when information is missing.

**15. What does "human-in-the-loop" mean in this project?**
It means the AI does not fully automate every decision — high-risk cases are deliberately routed to a human via the escalation branch, keeping a person responsible for sensitive outcomes like refunds or security issues.

**16. Where are credentials stored, and is anything exposed?**
The Groq API key is stored as an n8n credential (`Groq account 2`) and referenced only by name/ID in the workflow JSON — the actual key is never present in the exported file or in this documentation.

**17. What does `$json.output` represent?**
It's the parsed, structured result object returned by the AI Agent (via the Structured Output Parser), containing `category`, `urgency`, `sentiment`, `action`, `response`, and `reason`.

**18. Why does the escalation branch generate a `ticket_id`?**
To simulate creating a trackable support ticket record (`"TICKET-" + Date.now()`), which in a production system could be pushed to an external ticketing tool.

**19. What happens if the customer's message is missing required information (e.g., no order number)?**
Per the system prompt's rules, the agent should ask a clear follow-up question instead of guessing — this is demonstrated in the escalation test, where the agent asked for the order number and email.

**20. Is this workflow currently connected to a real ticketing system?**
No — `ticket_id` and `ticket_status` are generated as internal fields only; there is no live HTTP integration to an external helpdesk in the current implementation (documented as a limitation and future improvement).

**21. What are the main limitations of this project?**
Single LLM provider with no fallback, no real ticketing-system integration, static (non-queryable) FAQ knowledge, and no explicit customer identity verification step.

**22. What would you improve given more time?**
Integrate with a real ticketing platform via API, move the FAQ into a retrieval-augmented (RAG) knowledge base, add customer-tier-aware routing, and expand automated test coverage.

**23. Why does both branches (escalate/auto-resolve) converge into one Chat node?**
To keep a single source of truth for what is actually sent to the customer (`$json.customer_response`), regardless of which internal path was taken — simplifying the workflow and guaranteeing consistent response delivery.

**24. Could this architecture be reused for other domains?**
Yes — the same pattern (classify → structured output → route → respond) generalizes to e-commerce, SaaS support, banking, education, travel, or internal IT helpdesks, by adjusting the categories, escalation rules, and FAQ content.

**25. What real risk does this project mitigate compared to a plain chatbot?**
It prevents an AI from unilaterally handling money-related, legal, or security-sensitive requests by enforcing mandatory escalation rules — a plain chatbot without this logic could auto-respond to (and potentially mishandle) high-risk situations.
