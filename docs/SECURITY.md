# Security

**Docs:** [README](../README.md) · [Architecture](ARCHITECTURE.md) · [How It Works](HOW_IT_WORKS.md) · **Security** · [Setup](SETUP.md)

## At a glance

- The repository contains **sanitized workflow exports only**.
- **Production secrets are not versioned.**
- Credentials must be configured directly in **n8n / the environment**, never in
  the workflow JSON.
- **Real webhook URLs and infrastructure endpoints** (including the MCP endpoint)
  were removed.
- **Production execution data (`pinData`) and PII** were removed.
- **`.env` files must never be committed** — only `.env.example` (placeholders) is
  tracked.

## Scope of this repository

This repository contains **sanitized workflow definitions only**. It is safe to
publish and to share, but it is **not** a drop-in deployment: every environment
must supply its own configuration and credentials.

## What has been removed

The workflow JSON files were processed to strip everything that is
environment-specific or sensitive. None of the following is present in this
repository:

- API keys, tokens, secrets, passwords, authorization headers, bearer tokens
- OAuth client secrets, access tokens, refresh tokens
- n8n credential IDs and credential display names
- The n8n instance identifier (`instanceId`) and workflow `versionId`
- Production hostnames, base URLs, and IP addresses
- Production webhook URLs and the real MCP endpoint
- Google Drive folder IDs and Google Calendar IDs
- The Evolution API instance name and any real phone numbers
- All `pinData` (pinned execution data)
- Real execution payloads, WhatsApp message IDs, and cryptographic message
  material
- Customer names and any personally identifiable information
- Real customer messages
- Company-specific identity, pricing, services, and policies in the agent prompt
  (replaced with a neutral, fictional placeholder company)

Every removed value is represented by an explicit placeholder token, e.g.
`YOUR_N8N_BASE_URL`, `YOUR_WEBHOOK_ID`, `YOUR_MCP_ENDPOINT`,
`YOUR_GOOGLE_DRIVE_FOLDER_ID`, `YOUR_GOOGLE_CALENDAR_ID`,
`YOUR_EVOLUTION_INSTANCE`, `YOUR_SUPABASE_CREDENTIAL`, `YOUR_POSTGRES_CREDENTIAL`,
`YOUR_LLM_CREDENTIAL`, `YOUR_PHONE_NUMBER`, `YOUR_RESET_KEYWORD`.

The only identifiers left in the JSON are n8n's internal structural node
identifiers (`nodes[].id`, `assignments[].id`, `conditions[].id`), which are
required for the workflow graph to load and carry no secret or personal data.

## No credentials are versioned

- Credentials are **never** stored in the workflow JSON. In n8n, credentials live
  in the instance credential store and workflows only reference them by ID.
- In this repository those references are placeholders. After import you must
  create the credentials in your own n8n instance and re-link each node.
- `.env` and any `*.env` file are ignored by `.gitignore`. Only `.env.example`
  (placeholders only) is tracked.

## Configuration must come from the environment

Sensitive configuration — base URLs, instance names, folder and calendar IDs,
webhook IDs, keys — must be provided at deployment time through:

- the n8n **credential store** (for anything that authenticates), and
- **environment variables** / n8n expressions for non-secret but
  environment-specific values.

See `.env.example` for the full list of expected variables and
`docs/SETUP.md` for the provisioning steps.

## Rules for contributors

1. **Never** paste a secret, key, token, URL with an embedded token, real phone
   number, real customer message, or `pinData` into a workflow JSON.
2. **Always** export workflows from n8n with pinned data cleared before
   committing.
3. Re-run a secret scan before opening a pull request. Reject any diff that adds
   a real hostname, IP, credential ID, or personal data.
4. Keep placeholders in the `YOUR_*` format so they are easy to grep and
   obviously not real.
5. If a secret is ever committed by accident, treat it as compromised: rotate it
   immediately and purge it from history.

## Operational hardening (deployment side)

- Serve the WhatsApp provider and n8n over HTTPS only.
- Restrict the MCP endpoint and webhook URLs; treat their IDs as secrets.
- Use least-privilege service accounts for Google Drive and Google Calendar.
- Scope the Supabase key to only what the vector store and `leads` table need.
- Restrict who can trigger the conversation-reset keyword.

---

**Docs:** [README](../README.md) · [Architecture](ARCHITECTURE.md) · [How It Works](HOW_IT_WORKS.md) · **Security** · [Setup](SETUP.md)
