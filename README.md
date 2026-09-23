# Customer Support Agent — Complete Support Automation

AI-powered customer support automation built in **n8n**, using a Groq-hosted LLM AI Agent to classify customer messages, generate responses, and route requests between automatic resolution and human escalation.

> Assignment 06 — The AI School

## Objective
Build an AI-powered Customer Support Agent that receives customer messages, understands the request, classifies the issue, checks support information, generates an appropriate response, and decides whether the issue can be auto-resolved or must be escalated to a human.

## Problem Statement
Manually triaging every incoming support message is slow and inconsistent. Low-risk queries (password resets, order tracking) don't need a human, while high-risk ones (refunds, billing disputes, security issues) must not be auto-answered. This project builds an agent that makes that triage decision automatically and safely.

## Features
- Chat-based customer input (n8n Chat Trigger)
- Session-based conversation memory (last 10 messages)
- AI classification: category, urgency, sentiment
- Decision logic: `auto_resolve` vs `escalate`
- Structured JSON output (Structured Output Parser)
- FAQ/knowledge-base guided responses for common issues
- Automatic ticket creation on escalation (`ticket_id`, `ticket_status`)
- Guardrails against hallucinated policies, prices, or promises

## Workflow Overview
```
When chat message received → AI Agent (⇄ Groq Chat Model, Simple Memory, Structured Output Parser)
        → If (action == "escalate")
            ├── TRUE  → Edit Fields   (escalation record + ticket_id)
            └── FALSE → Edit Fields1  (auto-resolution record)
        → Chat (Send Message: customer_response)
```

## Technology Stack
- **n8n** — workflow orchestration
- **Groq API** — model `openai/gpt-oss-120b`
- **n8n LangChain nodes**: Chat Trigger, AI Agent, Groq Chat Model, Memory Buffer Window, Structured Output Parser, Chat (Send Message)
- **n8n core nodes**: IF, Set (Edit Fields)

## n8n Architecture
See `documentation/ARCHITECTURE.md` for the full node-by-node explanation and diagram.

## AI Agent Details
- Role: "AI Customer Support Agent" — analyzes each message and returns a structured classification + response.
- Classifies into 8 categories, 4 urgency levels, 4 sentiment levels, and 2 actions.
- Full system prompt reproduced in `documentation/AI_PROMPT_LIBRARY.md`.

## Groq Model
- Model: `openai/gpt-oss-120b`
- Connected to the AI Agent via the `ai_languageModel` input (`Groq Chat Model` node)
- Credential: referenced only by n8n credential name (`Groq account 2`) — no key exposed

## Conversation Memory
- Node: `Simple Memory` (`memoryBufferWindow`), context window length = 10 messages
- Keyed to the Chat Trigger's session ID, so the agent recalls prior turns in the same conversation

## Structured Output
- Node: `Structured Output Parser`, enforcing exactly these fields on every AI Agent response:
  `category`, `urgency`, `sentiment`, `action`, `response`, `reason`
- Raw parsed result is available at `$json.output`

## Auto-Resolution
- Triggered when `action == "auto_resolve"`
- `Edit Fields1` node builds: `ticket_status = "Auto-Resolved"`, `category`, `urgency`, `sentiment`, `customer_response`, `resolution_reason`

## Human Escalation
- Triggered when `action == "escalate"`
- `Edit Fields` node builds: `ticket_status = "Escalated"`, `category`, `urgency`, `sentiment`, `customer_response`, `escalation_reason`, `ticket_id = "TICKET-" + Date.now()`
- Escalation rules (from the system prompt): refund/cancellation involving money, financial disputes, duplicate/incorrect charges, payment problems, account security/hacking/unauthorized access, legal concerns, serious complaints, very negative sentiment, and any issue requiring a human decision

## FAQ Knowledge Base
Built into the AI Agent's system prompt, covering: password reset, order tracking, profile update, login problems, billing problems, and refund/cancellation guidance (guidance only — refunds/cancellations are always escalated).

## Testing
Two tests have been executed successfully against the live workflow. Full details in `documentation/TEST_CASES.md` and screenshots in `screenshots/`.

| Test | Input | Result |
|---|---|---|
| Auto-Resolve | "How do I reset my password?" | category=How-to, urgency=Low, sentiment=Neutral, action=auto_resolve |
| Escalation | "I was charged twice for the same order and I want my money back." | category=Billing, urgency=High, sentiment=Negative, action=escalate |

## How to Import the Workflow
1. Open n8n → Workflows → **Import from File**.
2. Select `workflow/Customer_Support_Agent.json`.
3. The workflow imports with all nodes and connections intact (unmodified from the working export).

## How to Configure Groq Credentials
1. In n8n, go to **Credentials → New → Groq API**.
2. Enter your own Groq API key (not included in this package for security).
3. Open the `Groq Chat Model` node and select your credential.

## How to Run the Workflow
1. Activate the workflow (or open it in the editor with the chat panel).
2. Open the chat panel from the `When chat message received` trigger.
3. Type a customer support message and send.
4. The agent classifies the message and returns a `customer_response` via the `Chat` node.

## Example Inputs & Expected Outputs
| Input | Expected Category | Expected Action |
|---|---|---|
| "How do I reset my password?" | How-to | auto_resolve |
| "I was charged twice for the same order and I want my money back." | Billing | escalate |
| "My account was hacked, someone logged in without me." | Account | escalate |
| "Where is my order?" | Order/Delivery | auto_resolve (unless tracking info missing → follow-up question) |

## Limitations
- Single LLM provider (Groq); no fallback model.
- No live integration with an actual ticketing system — `ticket_id`/`ticket_status` are generated fields only, not pushed to an external helpdesk.
- No authentication/verification of customer identity within the workflow itself.
- FAQ knowledge is static text inside the prompt, not a queryable external knowledge base.

## Future Improvements
- Connect `ticket_id`/`ticket_status` output to a real ticketing system (e.g., Zendesk, Freshdesk) via an HTTP Request node.
- Replace the in-prompt FAQ with a vector-store/RAG knowledge base for larger FAQ sets.
- Add customer-tier/priority as an input signal to refine routing.
- Add automated regression test suite covering more edge-case inputs.

## Project Structure
```
Customer_Support_Agent_Final/
├── README.md
├── LICENSE
├── SUBMISSION_CHECKLIST.md
├── workflow/
│   └── Customer_Support_Agent.json
├── documentation/
│   ├── PROJECT_REPORT.md
│   ├── ANALYSIS_QUESTIONS.md
│   ├── ARCHITECTURE.md
│   ├── AI_PROMPT_LIBRARY.md
│   ├── TEST_CASES.md
│   └── VIVA_QUESTIONS.md
├── presentation/
│   └── Customer_Support_Agent_Presentation.pptx
└── screenshots/
    ├── 02_auto_resolve_test.png
    ├── 03_escalation_test.png
    └── README.txt
```

## Security Note
No API keys, credential secrets, or tokens are included anywhere in this package. The Groq credential is referenced only by its n8n credential name (`Groq account 2`); it must be configured separately by whoever imports the workflow.
## Live Demo

Customer Support Agent:
[Open Live Customer Support Chat](https://shreesh636.app.n8n.cloud/webhook/d1e7cd64-8abf-49ec-bebd-76d422bb6a45/chat)

Github Repositoru link-https://github.com/Shreesh636/customer-support-agent.git
