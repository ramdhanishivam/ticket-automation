# Ticket Automation

## Onboarding Automation & AI Concierge – Personal Project Blueprint

**A TypeScript/Node.js Hybrid System**  
Automates ticket-based onboarding flows and provides a conversational AI assistant for new joiners.

---

## 1. Project Vision

Eliminate the repetitive, multi-platform drudgery that every new employee faces when they receive their company email.  
- **Core Automation** (deterministic, event-driven): Listens for a new hire event, generates role/team‑based tickets in ServiceNow/Jira, sends a welcome email with buddy info and learning resources.
- **AI Conversational Concierge** (LLM + RAG + tool use): A chatbot that answers new‑joiner questions 24/7, retrieves ticket statuses, guides learning paths, and reduces “first‑week anxiety.”

---

## 2. High‑Level Architecture

```
[HR System / Webhook]
            │
            ▼
    ┌───────────────────────────┐
    │       Automation API      │ Node.js + TypeScript
    │        (Event Listener)   │
    └───────────┬───────────────┘
                │
                ▼
    ┌───────────────────────────┐
    │     Onboarding Orchestrator│
    │ - Reads Role-Team Matrix   │
    │ - Creates tickets          │ Calls ServiceNow / Jira REST APIs
    │ - Sends Welcome Email      │
    │ - Logs to DB               │
    └───────────┬───────────────┘
                │
                ▼
    ┌───────────────────────────┐
    │       AI Concierge Service │ Node.js + TypeScript
    │        (Chatbot Backend)  │ Uses LLM, Vector DB, Ticket API
    │ - Answers queries          │
    │ - Checks ticket status     │
    │ - Recommends resources     │
    └───────────────────────────┘
                ▲
                │ (WebSocket/HTTP)
                ▼
    ┌───────────────────────────┐
    │        Chat Interface      │
    │   (Slack Bot/Web Chat)     │
    └───────────────────────────┘
```

The AI layer does **not** replace ticket creation; it sits on top to provide the conversational interface. The deterministic pipeline is the single source of truth for ticket generation.

---

## 3. Technology Stack (TypeScript & Node.js)

| Layer                | Technology                                                                     |
| -------------------- | ------------------------------------------------------------------------------ |
| Runtime              | Node.js 20+ (LTS)                                                             |
| Language             | TypeScript (strict mode)                                                      |
| API Framework        | Express or Fastify (performance)                                              |
| Background Jobs      | BullMQ (Redis‑based queues for ticket creation, email sending)                |
| Database             | PostgreSQL + Prisma ORM (stores employee records, ticket bundles, bot session)|
| Vector Store (RAG)   | ChromaDB (open‑source) or Pinecone (managed)                                  |
| LLM/AI Library       | OpenAI SDK (GPT‑4o mini) or LangChain.js for complex chains                   |
| Embedding Model      | `text-embedding-3-small` (OpenAI)                                             |
| Ticketing Integration| `axios` to call ServiceNow REST, `jira.js` or plain fetch for Jira Cloud      |
| Email                | Nodemailer + SMTP, or SendGrid API                                            |
| Chat Interface       | Slack Bolt Framework or a simple Next.js web chat (if you want a frontend)    |
| Configuration        | Environment variables, Zod for validation                                     |

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
```

### 4.3 Ticket Creation Flow

1. Resolve target system (ServiceNow or Jira) and endpoint.
2. Build payload with employeeName, managerEmail, predefined approval policy ID, and a clear description (auto‑generated).
3. Submit request via HTTP.
4. Store returned ticket ID in a TicketBundle DB record linked to the employee.
5. Queue a retry on failure.

### 4.4 Email Service

Send one consolidated welcome email using a template:
- Greeting
- Buddy name and intro
- Structured list of learning resources
- Invite link to the AI Concierge chatbot
- Ticket‑bundle reference (for the chatbot to query)

---

## 5. AI Conversational Concierge Module

### 5.1 Capabilities
1. **Ticket Status Check**: “When will my software be ready?”  
2. **Personalized Q&A**: “How do I connect to the ClientX VPN?”  
3. **Learning Guidance**: “What should I study first as a Frontend Dev?”  
4. **Buddy Scheduling Info**: “When is Priya available?”  
5. **Company Facts**: “What does ClientX actually do?”  

### 5.2 AI Architecture

- **Agent Tools:**
  1. getOnboardingTicketStatus(employeeId) – queries your DB.
  2. searchKnowledgeBase(query) – queries ChromaDB with vector similarity search.
  3. getLearningPath(employeeId) – retrieves learning path sequence.
  4. getBuddyInfo(employeeId) – retrieves buddy availability info.

- **Decision Making**: LLM integrates relevant tools for generating user-friendly answers.

---

For future integration plan, reference details like "MVP -> Implementation" *(Phase Section)* Steps Incrementally.

