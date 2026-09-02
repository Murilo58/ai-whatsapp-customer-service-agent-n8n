# AI WhatsApp Customer Service Agent

> Production-oriented AI agent architecture for automated customer service on WhatsApp.

A modular, reusable automation built on **n8n** that delivers autonomous customer
service over WhatsApp: it understands text and voice messages, answers from a
curated knowledge base (RAG), manages appointments through Google Calendar, keeps
per-contact conversational memory, tracks leads, and escalates to a human agent
when the situation requires it.

The solution is organized as **four independent workflows** that communicate
through well-defined interfaces (webhook, MCP endpoint, sub-workflow calls, shared
database), so each part can be deployed, versioned, and scaled on its own.

---

## Key capabilities

| Area | What it does |
|------|--------------|
| **Automated WhatsApp service** | Inbound messages are received through a webhook and answered automatically, with a natural "typing…" presence and message-splitting for a human-like cadence. |
| **Evolution API integration** | WhatsApp connectivity (receive events, send messages, send presence) is handled through the Evolution API messaging layer. |
| **AI Agent** | A tool-calling LLM agent orchestrates every turn: classifies intent, decides which tool to use, and produces the customer-facing reply. |
| **RAG knowledge base** | Company information (services, pricing, hours, policies, FAQ) is retrieved from a vector store before the agent answers questions about the business. |
| **Google Drive ingestion** | A dedicated workflow watches a Drive folder; new or updated documents are automatically chunked, embedded, and upserted into the knowledge base. |
| **Vector Store / Supabase** | Embeddings and document chunks are stored in a Supabase (pgvector) vector store with metadata for source traceability. |
| **Embeddings** | Documents and queries are embedded through a configurable embeddings provider. |
| **Persistent memory (PostgreSQL)** | Conversation history is stored per contact in PostgreSQL, so context survives across messages and sessions. |
| **Lead management** | Every contact is upserted into a `leads` table with name, phone, message count, status, tags, and last-interaction timestamp. |
| **Text & audio processing** | A router separates text messages from voice messages and handles each path appropriately. |
| **Audio transcription** | Voice notes are converted from base64 to binary and transcribed by a speech-capable model before entering the agent. |
| **MCP** | Scheduling capabilities are exposed as an **MCP server** (n8n MCP trigger) and consumed by the main workflow through an **MCP client tool**, decoupling the agent from the calendar implementation. |
| **Google Calendar integration** | The MCP tools workflow can **list, create, update, and cancel** calendar events on the operation's scheduling calendar. |
| **Human handoff** | A sub-workflow tags the lead for human service and notifies the operations team with the reason and full context. |
| **Anti-hallucination rules** | The agent is instructed to never invent prices, services, hours, or availability, and to consult the knowledge base or defer to the team when unsure. |
| **Confirmation before critical actions** | No appointment is created, changed, or deleted without an explicit customer confirmation and a summary shown first. |
| **Modular architecture** | Four workflows: main orchestrator, RAG ingestion, calendar MCP tools, human handoff. |

---

## Workflows

| File | Role |
|------|------|
| [`workflows/ai-agent-main.json`](workflows/ai-agent-main.json) | Main orchestrator: webhook intake, lead management, message routing, audio transcription, buffering/debounce, AI Agent, response delivery. |
| [`workflows/rag-knowledge-base.json`](workflows/rag-knowledge-base.json) | Knowledge base ingestion: watches Google Drive, extracts text, chunks, embeds, and upserts into the Supabase vector store. |
| [`workflows/calendar-mcp-tools.json`](workflows/calendar-mcp-tools.json) | MCP server exposing Google Calendar tools (search, create, update, delete events). |
| [`workflows/human-handoff.json`](workflows/human-handoff.json) | Sub-workflow: tags the lead for human service and notifies the team. |

A detailed description and a Mermaid diagram are in
[`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

---

## Repository contents

```
.
├── README.md
├── .gitignore
├── .env.example
├── workflows/
│   ├── ai-agent-main.json
│   ├── rag-knowledge-base.json
│   ├── calendar-mcp-tools.json
│   └── human-handoff.json
└── docs/
    ├── ARCHITECTURE.md
    ├── SECURITY.md
    └── SETUP.md
```

> This repository ships **sanitized** workflow definitions only. No credentials,
> endpoints, identifiers, or production data are versioned. See
> [`docs/SECURITY.md`](docs/SECURITY.md).

---

## Getting started

1. Read [`docs/SETUP.md`](docs/SETUP.md) for prerequisites and the provisioning
   checklist.
2. Copy `.env.example` and provide your own values through the environment / n8n
   credential store — never inside the workflow JSON.
3. Import the four workflows from `workflows/` into your n8n instance.
4. Reconnect each node's credentials and replace the `YOUR_*` placeholders
   (webhook IDs, MCP endpoint, calendar ID, Drive folder ID, instance name).
5. Activate the workflows and point your WhatsApp provider webhook at the main
   workflow.

---

## Tech stack

- **Orchestration:** n8n (LangChain nodes)
- **Messaging:** Evolution API (or an equivalent WhatsApp API)
- **LLM & transcription:** configurable provider (chat + audio-capable model)
- **Embeddings:** configurable embeddings provider
- **Vector store:** Supabase / pgvector
- **Memory & lead store:** PostgreSQL (Supabase-compatible)
- **Buffer / debounce:** Redis
- **Scheduling:** Google Calendar via MCP
- **Knowledge source:** Google Drive
