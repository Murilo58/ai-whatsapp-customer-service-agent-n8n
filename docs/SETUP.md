# Setup

This guide is intentionally generic. It lists what the architecture needs and how
the pieces fit together, not vendor-specific instructions.

## Prerequisites

| Component | Purpose | Notes |
|-----------|---------|-------|
| **n8n** | Runs all four workflows. | A recent version with the LangChain / AI nodes and the MCP trigger + MCP client nodes. Self-hosted or managed. |
| **Evolution API** (or equivalent WhatsApp API) | Sends/receives WhatsApp messages and presence. | Any provider that can POST inbound events to a webhook and expose a send-message / send-presence API. |
| **PostgreSQL** | Conversational memory (per-contact chat history) and the `leads` table. | Supabase's Postgres works; any managed Postgres is fine. |
| **Supabase** | Vector store (pgvector) for the knowledge base. | Requires the `documents` table and a `match_documents` similarity function. |
| **Redis** | Short-lived message buffer / debounce. | Any Redis-compatible instance. |
| **Google Drive** | Source of knowledge-base documents. | One folder the ingestion workflow watches. |
| **Google Calendar** | Appointment storage. | One calendar the MCP tools operate on. |
| **LLM provider** | Chat model for the agent + audio-capable model for transcription. | Configurable; the workflows reference it through an n8n credential. |
| **Embeddings provider** | Turns documents and queries into vectors. | Can be the same vendor as the LLM or a different one. |

## Provisioning checklist

1. **Databases**
   - Create the PostgreSQL database. n8n's Postgres chat-memory node will create
     its history table on first use; create the `leads` table with at least:
     `id`, `Telefone`, `Nome`, `status`, `Tags`, `message_count`,
     `last_interaction`, `created_at`.
   - In Supabase, enable `pgvector`, create the `documents` table and the
     `match_documents` function used by the vector store node.

2. **Redis** — provision an instance and note host / port / password.

3. **Google Cloud**
   - Create OAuth2 credentials for Drive and for Calendar (or a service account
     with domain-wide delegation, depending on your setup).
   - Create/choose the knowledge-base **Drive folder** and the scheduling
     **Calendar**; record their IDs for the placeholders.

4. **WhatsApp provider**
   - Provision an instance/number.
   - Configure its webhook to POST inbound events to the main workflow's webhook
     URL.

5. **LLM & embeddings** — obtain API access and pick the chat model, the
   audio/transcription model, and the embeddings model.

## Configure

1. Copy the example environment file and fill in your own values:
   ```
   cp .env.example .env
   ```
   Keep `.env` out of version control (already covered by `.gitignore`).

2. In n8n, create one credential per external system:
   - WhatsApp / Evolution API
   - PostgreSQL
   - Supabase
   - Redis
   - Google Drive (OAuth2)
   - Google Calendar (OAuth2)
   - LLM provider
   - Embeddings provider

3. Import the workflows from `workflows/`:
   - `ai-agent-main.json`
   - `rag-knowledge-base.json`
   - `calendar-mcp-tools.json`
   - `human-handoff.json`

4. In every node, replace the `YOUR_*` placeholders and re-link credentials:
   - `YOUR_WEBHOOK_ID` — regenerate the webhook in the main workflow.
   - `YOUR_MCP_ENDPOINT_ID` / `YOUR_MCP_ENDPOINT` — regenerate the MCP trigger in
     `calendar-mcp-tools.json`, then point the MCP client node in the main
     workflow at the resulting URL.
   - `YOUR_WAIT_WEBHOOK_ID` / `YOUR_WAIT_WEBHOOK_ID_2` — recreated automatically
     by n8n for the Wait nodes.
   - `YOUR_GOOGLE_DRIVE_FOLDER_ID`, `YOUR_GOOGLE_CALENDAR_ID`,
     `YOUR_EVOLUTION_INSTANCE`, `YOUR_PHONE_NUMBER`.
   - `YOUR_SUBWORKFLOW_ID` — set to the imported `human-handoff` workflow.
   - `YOUR_RESET_KEYWORD` — choose a non-obvious keyword and restrict who knows
     it.
   - In the AI Agent system prompt, replace the fictional company/assistant name
     with your own and adjust the flows/policies to your operation.

## Bring-up order

1. Activate `calendar-mcp-tools.json`; copy its MCP endpoint URL.
2. Activate `rag-knowledge-base.json`; drop a document in the Drive folder and
   confirm chunks land in `documents`.
3. Activate `human-handoff.json`.
4. In `ai-agent-main.json`, wire the MCP client and the human-handoff tool, then
   activate it.
5. Point the WhatsApp webhook at the main workflow and send a test message.

## Smoke tests

- Send a text question answerable from the knowledge base → grounded answer, no
  invented details.
- Send a voice note → transcription then a relevant reply.
- Ask to book, then confirm → event appears in Google Calendar after an explicit
  confirmation.
- Ask for a human → lead gets the `Atendimento Humano` tag and the team receives
  the notification; automatic replies stop.
- Send the reset keyword → stored conversation memory for that contact is
  cleared.
