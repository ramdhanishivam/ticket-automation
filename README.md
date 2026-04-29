# ticket-automation

# Onboarding Automation & AI Concierge – Personal Project Blueprint

**A TypeScript/Node.js Hybrid System**  
Automates ticket-based onboarding flows and provides a conversational AI assistant for new joiners.

---

## 1. Project Vision

Eliminate the repetitive, multi-platform drudgery that every new employee faces when they receive their company email.  
- **Core Automation** (deterministic, event-driven): Listens for a new hire event, generates role/team‑based tickets in ServiceNow/Jira, sends a welcome email with buddy info and learning resources.  
- **AI Conversational Concierge** (LLM + RAG + Tool use): A chatbot that answers new‑joiner questions 24/7, retrieves ticket statuses, guides learning paths, and reduces “first‑week anxiety”—all without replacing the essential human fulfilment steps.

---

## 2. High‑Level Architecture
[HR System / Webhook]
│
▼
┌───────────────────────────┐
│ Automation API │ Node.js + TypeScript
│ (Event Listener) │
└─────────┬─────────────────┘
│
▼
┌───────────────────────────┐
│ Onboarding Orchestrator │
│ - Reads Role-Team Matrix│
│ - Creates tickets │ Calls ServiceNow / Jira REST APIs
│ - Sends Welcome Email │
│ - Logs to DB │
└─────────┬─────────────────┘
│
▼
┌───────────────────────────┐
│ AI Concierge Service │ Node.js + TypeScript
│ (Chatbot Backend) │ Uses LLM, Vector DB, Ticket API
│ - Answers queries │
│ - Checks ticket status │
│ - Recommends resources │
└───────────────────────────┘
▲
│ (WebSocket/HTTP)
▼
┌───────────────────────────┐
│ Chat Interface │
│ (Slack Bot/Web Chat) │
└───────────────────────────┘


The AI layer does **not** replace ticket creation; it sits on top to provide the conversational interface. The deterministic pipeline is the single source of truth for ticket generation.

---

## 3. Technology Stack (TypeScript & Node.js)

| Layer                | Technology                                                                     |
| -------------------- | ------------------------------------------------------------------------------ |
| Runtime              | Node.js 20+ (LTS)                                                              |
| Language             | TypeScript (strict mode)                                                       |
| API Framework        | Express or Fastify (performance)                                               |
| Background Jobs      | BullMQ (Redis‑based queues for ticket creation, email sending)                 |
| Database             | PostgreSQL + Prisma ORM (stores employee records, ticket bundles, bot session) |
| Vector Store (RAG)   | ChromaDB (open‑source) or Pinecone (managed)                                   |
| LLM/AI Library       | OpenAI SDK (GPT‑4o mini) or LangChain.js for complex chains                    |
| Embedding Model      | `text-embedding-3-small` (OpenAI)                                              |
| Ticketing Integration| `axios` to call ServiceNow REST, `jira.js` or plain fetch for Jira Cloud       |
| Email                | Nodemailer + SMTP, or SendGrid API                                             |
| Chat Interface       | Slack Bolt Framework or a simple Next.js web chat (if you want a frontend)     |
| Configuration        | Environment variables, Zod for validation                                      |

---

## 4. Core Automation Module (Deterministic Pipeline)

### 4.1 Trigger
- A webhook from HR system (e.g., `POST /webhook/new-hire`) containing: `employeeId`, `name`, `email`, `role`, `team`, `managerEmail`, `startDate`.
- Or a cron job that polls a mock JSON/DB for “pending” hires.

### 4.2 Role‑Team Matrix
A configuration file or DB table mapping `role + team` to:
```json
{
  "role": "Java FS Developer",
  "team": "ClientX",
  "software": [
    {"name": "IntelliJ Ultimate", "system": "ServiceNow", "type": "software_install"},
    {"name": "Postman", "system": "ServiceNow", "type": "software_install"},
    {"name": "ClientX VPN", "system": "Jira_ClientX", "type": "access_request"}
  ],
  "access_groups": ["github:clientx-falcon", "aws:clientx-sandbox"],
  "buddy": "priya.sharma@company.com",
  "learning_resources": [
    {"title": "Product Overview", "url": "https://wiki.company.com/product-overview"},
    {"title": "Dev Setup Guide", "url": "https://wiki.company.com/dev-setup"}
  ]
}

his matrix is the single source of truth; it is maintained manually (or, optionally, by a future AI‑curator tool).

4.3 Ticket Creation Flow
For each item in the list:

Resolve target system (ServiceNow or Jira) and endpoint.

Build payload with employeeName, managerEmail, predefined approval policy ID, and a clear description (auto‑generated).

Submit request via HTTP.

Store returned ticket ID in a TicketBundle DB record linked to the employee.

Queue a retry on failure.

4.4 Email Service
Send one consolidated welcome email using a template:

Greeting

Buddy name and intro

Structured list of learning resources

Invite link to the AI Concierge chatbot

Ticket‑bundle reference (for the chatbot to query)

5. AI Conversational Concierge Module
This is the “intelligence layer” – a chatbot that understands natural language and can reason over your internal knowledge base and act on your ticket API.

5.1 Capabilities
Ticket Status Check: “When will my software be ready?”

Personalized Q&A: “How do I connect to the ClientX VPN?”

Learning Guidance: “What should I study first as a Frontend Dev?”

Buddy Scheduling Info: “When is Priya available?”

Company Facts: “What does ClientX actually do?”

5.2 AI Architecture
User message arrives via Slack/web chat.

The message is processed by an agent loop (using LangChain’s AgentExecutor or your own implementation with OpenAI function calling).

The agent has access to these tools (TypeScript functions):

getOnboardingTicketStatus(employeeId) – queries your DB.

searchKnowledgeBase(query) – queries ChromaDB with vector similarity search, returning top‑3 chunks.

getLearningPath(employeeId) – returns the role‑based sequence of resources.

getBuddyInfo(employeeId) – returns buddy name, email, and next available slot (if integrated with calendar).

The LLM decides when to call which tool, then synthesises a friendly answer.

5.3 Data Pipeline for Knowledge Base
Ingest all onboarding‑related documents (PDFs, markdown, wiki exports) into a vector store.

Use a simple script that splits documents into chunks and stores embeddings.

Refresh periodically (or when documents change) – this is a separate ingest.ts utility.

5.4 Security & Guardrails
The bot authenticates the user (via Slack user ID or a simple login token stored in DB) and can only access data belonging to that employee.

Prompt restricts answers to onboarding topics; out‑of‑scope queries are politely declined.

All bot interactions are logged for improvement.

6. Implementation Roadmap (MVP -> Full)
Phase 1: Foundation & Core Automation (Week 1–2)
Initialize Node.js/TypeScript project with Express/Fastify.

Set up PostgreSQL and Prisma schema (Employee, RoleMatrix, TicketBundle).

Create the webhook endpoint to receive new hire events.

Build the ticket‑creation service (start with ServiceNow mock, then real).

Implement Nodemailer welcome email.

Deliverable: After trigger, tickets are created and email sent, all logged in DB.

Phase 2: AI Assistant Backend (Week 3–4)
Integrate OpenAI API (or Anthropic) and set up function‑calling definitions.

Build the knowledge base ingestion pipeline (ChromaDB + OpenAI embeddings).

Implement the agent tools (getOnboardingTicketStatus, searchKnowledgeBase, etc.).

Create the /chat endpoint that processes messages and returns AI responses.

Test in a simple terminal‑based REPL first.

Phase 3: Chat Interface & Integration (Week 5)
Build a Slack bot using @slack/bolt (or a lightweight web chat UI with Next.js if you prefer full‑stack).

Connect the bot to your chat backend.

Add authentication (Slack user ID → employee email mapping).

End‑to‑end test: HR triggers webhook → email sent → user opens Slack → asks “What’s my buddy’s name?” → bot replies with correct info.

Phase 4: Polish & Production Like (Week 6)
Add BullMQ queues for all long‑running tasks (ticket creation, email).

Implement retry logic and dead‑letter queue.

Write unit and integration tests (Jest/Vitest).

Dockerize the whole application (separate services: api, worker, bot).

Write a comprehensive README and a short demo video script.

7. Project Structure (Suggested)
text
onboarding-automation/
├── packages/
│   ├── core/                    # Shared types, utils, DB client
│   ├── automation/              # Webhook handler, ticket creator, email
│   ├── ai-assistant/            # Chat endpoint, agent logic, tools
│   └── ingestion/               # Document ingestion for vector store
├── docker-compose.yml           # PostgreSQL, Redis, ChromaDB
├── .env.example
├── package.json                 # Monorepo root (using turborepo or npm workspaces)
└── README.md
8. Key Differentiators for Your AI Skills Portfolio
Function Calling / Tool Use: You demonstrate an LLM that not just chats, but actively retrieves real‑time data from a database (ticket status) and a knowledge base.

RAG with Vector Search: You show practical handling of company documents, chunking, embedding, and semantic retrieval.

Hybrid System Design: You combine deterministic business logic with probabilistic AI in a safe, maintainable architecture.

Real Business Problem: This is not a toy; you solve a well‑known enterprise pain point, making your project highly relatable in interviews.

TypeScript Full‑Stack: You prove you can build a production‑grade Node.js app with strict typing, background jobs, and external integrations.


his matrix is the single source of truth; it is maintained manually (or, optionally, by a future AI‑curator tool).

4.3 Ticket Creation Flow
For each item in the list:

Resolve target system (ServiceNow or Jira) and endpoint.

Build payload with employeeName, managerEmail, predefined approval policy ID, and a clear description (auto‑generated).

Submit request via HTTP.

Store returned ticket ID in a TicketBundle DB record linked to the employee.

Queue a retry on failure.

4.4 Email Service
Send one consolidated welcome email using a template:

Greeting

Buddy name and intro

Structured list of learning resources

Invite link to the AI Concierge chatbot

Ticket‑bundle reference (for the chatbot to query)

5. AI Conversational Concierge Module
This is the “intelligence layer” – a chatbot that understands natural language and can reason over your internal knowledge base and act on your ticket API.

5.1 Capabilities
Ticket Status Check: “When will my software be ready?”

Personalized Q&A: “How do I connect to the ClientX VPN?”

Learning Guidance: “What should I study first as a Frontend Dev?”

Buddy Scheduling Info: “When is Priya available?”

Company Facts: “What does ClientX actually do?”

5.2 AI Architecture
User message arrives via Slack/web chat.

The message is processed by an agent loop (using LangChain’s AgentExecutor or your own implementation with OpenAI function calling).

The agent has access to these tools (TypeScript functions):

getOnboardingTicketStatus(employeeId) – queries your DB.

searchKnowledgeBase(query) – queries ChromaDB with vector similarity search, returning top‑3 chunks.

getLearningPath(employeeId) – returns the role‑based sequence of resources.

getBuddyInfo(employeeId) – returns buddy name, email, and next available slot (if integrated with calendar).

The LLM decides when to call which tool, then synthesises a friendly answer.

5.3 Data Pipeline for Knowledge Base
Ingest all onboarding‑related documents (PDFs, markdown, wiki exports) into a vector store.

Use a simple script that splits documents into chunks and stores embeddings.

Refresh periodically (or when documents change) – this is a separate ingest.ts utility.

5.4 Security & Guardrails
The bot authenticates the user (via Slack user ID or a simple login token stored in DB) and can only access data belonging to that employee.

Prompt restricts answers to onboarding topics; out‑of‑scope queries are politely declined.

All bot interactions are logged for improvement.

6. Implementation Roadmap (MVP -> Full)
Phase 1: Foundation & Core Automation (Week 1–2)
Initialize Node.js/TypeScript project with Express/Fastify.

Set up PostgreSQL and Prisma schema (Employee, RoleMatrix, TicketBundle).

Create the webhook endpoint to receive new hire events.

Build the ticket‑creation service (start with ServiceNow mock, then real).

Implement Nodemailer welcome email.

Deliverable: After trigger, tickets are created and email sent, all logged in DB.

Phase 2: AI Assistant Backend (Week 3–4)
Integrate OpenAI API (or Anthropic) and set up function‑calling definitions.

Build the knowledge base ingestion pipeline (ChromaDB + OpenAI embeddings).

Implement the agent tools (getOnboardingTicketStatus, searchKnowledgeBase, etc.).

Create the /chat endpoint that processes messages and returns AI responses.

Test in a simple terminal‑based REPL first.

Phase 3: Chat Interface & Integration (Week 5)
Build a Slack bot using @slack/bolt (or a lightweight web chat UI with Next.js if you prefer full‑stack).

Connect the bot to your chat backend.

Add authentication (Slack user ID → employee email mapping).

End‑to‑end test: HR triggers webhook → email sent → user opens Slack → asks “What’s my buddy’s name?” → bot replies with correct info.

Phase 4: Polish & Production Like (Week 6)
Add BullMQ queues for all long‑running tasks (ticket creation, email).

Implement retry logic and dead‑letter queue.

Write unit and integration tests (Jest/Vitest).

Dockerize the whole application (separate services: api, worker, bot).

Write a comprehensive README and a short demo video script.

7. Project Structure (Suggested)
text
onboarding-automation/
├── packages/
│   ├── core/                    # Shared types, utils, DB client
│   ├── automation/              # Webhook handler, ticket creator, email
│   ├── ai-assistant/            # Chat endpoint, agent logic, tools
│   └── ingestion/               # Document ingestion for vector store
├── docker-compose.yml           # PostgreSQL, Redis, ChromaDB
├── .env.example
├── package.json                 # Monorepo root (using turborepo or npm workspaces)
└── README.md
8. Key Differentiators for Your AI Skills Portfolio
Function Calling / Tool Use: You demonstrate an LLM that not just chats, but actively retrieves real‑time data from a database (ticket status) and a knowledge base.

RAG with Vector Search: You show practical handling of company documents, chunking, embedding, and semantic retrieval.

Hybrid System Design: You combine deterministic business logic with probabilistic AI in a safe, maintainable architecture.

Real Business Problem: This is not a toy; you solve a well‑known enterprise pain point, making your project highly relatable in interviews.

TypeScript Full‑Stack: You prove you can build a production‑grade Node.js app with strict typing, background jobs, and external integrations.

9. Future Expansion Ideas
AI Curator for Role Matrix: Periodically scan ticket history and suggest updates to the static software list.

Calendar Integration: Let the bot actually schedule buddy calls or IT sessions.

Manager Dashboard: A UI showing onboarding progress for all direct reports.

Multi‑language Support: Localize the concierge for global teams.

10. Getting Started – Immediate Next Step
Start by scaffolding the monorepo and setting up a simple Express API with a POST /webhook/new-hire endpoint. Log the payload, then gradually wire in the ticket and email logic. The AI assistant can be developed in parallel once the DB schema is ready.

Good luck building your standout AI‑augmented project!

text

This document should give you a clear, actionable map. If you need any section detailed further (like the agent tool definitions or Slack bot setup), let me know.
