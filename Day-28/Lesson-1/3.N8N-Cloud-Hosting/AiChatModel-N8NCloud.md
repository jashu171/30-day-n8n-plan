# n8n AI Chat Bot Workflow — n8n Cloud (Recommended for Production)

This guide explains how to deploy the **AI chat workflow** on **n8n Cloud** and expose a **stable, secure production webhook URL**. Using n8n Cloud is **good practice** for production because it provides **managed SSL**, **availability**, **security features**, and **clear separation** from developer laptops.

---

## What you will build
A workflow that receives a chat message via a **public production URL** on n8n Cloud, processes it through an **AI Agent** backed by **Google Gemini** with **Simple Memory**, and emails the result via **Gmail**.

```
Public (n8n Cloud URL) ──▶ When chat message received ──▶ AI Agent ──▶ Gmail: Send a message
                                              ▲                ▲
                          Google Gemini Chat Model ────────────┘
                          Simple Memory ───────────────────────┘
```

Screenshots live in `./images`:
- `./images/n8ncloud-dashboard.png` — n8n Cloud workspace
- `./images/n8n-workflow-cloud.png` — nodes wired up in n8n Cloud
- `./images/webhook-prod-url.png` — production webhook URL

---

## Why n8n Cloud is good practice for production
- **Stable, HTTPS production URL** with managed TLS certificates.
- **Uptime & scaling** handled by the platform instead of a developer machine.
- **Credential security** via n8n’s built-in credential manager.
- **Access control**: manage who can view/edit workflows.
- **Separation of environments**: keep **dev** and **prod** in separate **workspaces**.
- **Observability**: run history, execution logs, and error notifications.

---

## Prerequisites
- An **n8n Cloud** account and a **workspace** (e.g. `https://<your-workspace>.n8n.cloud`).
- **Google Gemini** (Google AI Studio) access for API keys/credentials.
- **Gmail OAuth2** credentials (client ID/secret) for sending email.
- The workflow JSON (or follow the minimal node snippets in the appendix).

> Tip: Use **separate workspaces** for **dev** and **prod**. Only the prod workspace should receive external traffic from end users.

---

## 1) Create/Select your n8n Cloud workspace
1. Sign in to n8n Cloud and open your workspace (URL looks like `https://<your-workspace>.n8n.cloud`).
2. In the left sidebar, click **Workflows** → **New** to create a fresh workflow, or **Import from File** if you already have a JSON export.

---

## 2) Import or build the workflow
If you have a JSON file, import it. Otherwise, build these nodes:

- **When chat message received** (Trigger)
- **AI Agent**
- **Google Gemini Chat Model**
- **Simple Memory**
- **Gmail: Send a message**

> See **Appendix A** for minimal JSON snippets you can paste into a new workflow and then wire the connections visually.

---

## 3) Configure credentials (Cloud-safe)

### 3.1 Google Gemini (LLM)
1. Create a **Google Gemini** (Google AI Studio) API key if you don’t have one.
2. In n8n Cloud: **Credentials** → **New** → pick the **Google Gemini** credential type used by your LLM node.
3. Paste the key and save.
4. In the **Google Gemini Chat Model** node, select this credential.

### 3.2 Gmail OAuth2
1. In **Google Cloud Console**, create **OAuth 2.0 Client** credentials (Web application).
2. In n8n Cloud, create a new **Gmail OAuth2** credential. n8n shows you an **OAuth Redirect URL** unique to your workspace.
3. Copy that redirect URL back into your **Google Cloud Console** OAuth client configuration.
4. Finish the OAuth handshake in n8n and select this credential on the **Gmail: Send a message** node.

> Keep client secrets **only in n8n credentials**. Do **not** hardcode secrets in workflow expressions.

---

## 4) Set the webhook to Production mode
1. Open **When chat message received**.
2. Ensure the node is **Public** (so it can receive external requests).
3. Save the workflow.
4. **Activate** the workflow (toggle to “Active”). Cloud provides two URLs:
   - **Test URL** (non-production): often `.../webhook-test/...`
   - **Production URL**: `https://<your-workspace>.n8n.cloud/webhook/<id>`

Use the **Production URL** for real traffic and external integrations.

**Example Production URL:**
```
https://example-team.n8n.cloud/webhook/d400435f-44fd-4587-8cff-f74848b47cb2
```

---

## 5) Test the production webhook
Use **curl** with the Production URL (replace with your node’s ID/path):

```bash
curl -X POST "https://example-team.n8n.cloud/webhook/d400435f-44fd-4587-8cff-f74848b47cb2"   -H "Content-Type: application/json"   -d '{
        "chatInput": "Hello from n8n Cloud"
      }'
```

Expected behavior (with the example wiring):
- The **AI Agent** processes the message with **Google Gemini** + **Simple Memory**.
- **Gmail** sends an email to your test inbox with both input and AI output.

---

## 6) Production hardening checklist
- **Separate workspaces**: dev vs prod, different credentials and URLs.
- **Principle of least privilege**: scope Gmail and any other credentials to the minimum required.
- **Rotate credentials** periodically.
- **Audit & logs**: enable error notifications; review failed executions.
- **Rate limiting**: apply throttling or guards in the workflow if you expect spikes.
- **Validation**: validate incoming request bodies and reject unexpected shapes.
- **Auth**: if appropriate, require a secret or token for the trigger (e.g., header-based key).

---

## Troubleshooting
- **401/403**: Check if the node is public (or your auth header/key is correct).
- **404**: Confirm you are calling the **Production URL** for an **Active** workflow.
- **OAuth errors (Gmail)**: Ensure the **OAuth Redirect URL** in Google Cloud Console **exactly matches** the one shown by n8n Cloud.
- **Emails not sending**: Verify the **Gmail: Send a message** node credential and recipient.
- **LLM errors**: Verify the Gemini credential and usage limits.

---

## Appendix A — Minimal node JSON snippets

**When chat message received**
```json
{{
  "name": "When chat message received",
  "type": "@n8n/n8n-nodes-langchain.chatTrigger",
  "typeVersion": 1.3,
  "parameters": {{
    "public": true,
    "options": {{}}
  }}
}}
```

**AI Agent**
```json
{{
  "name": "AI Agent",
  "type": "@n8n/n8n-nodes-langchain.agent",
  "typeVersion": 2.2,
  "parameters": {{
    "options": {{}}
  }}
}}
```

**Google Gemini Chat Model**
```json
{{
  "name": "Google Gemini Chat Model",
  "type": "@n8n/n8n-nodes-langchain.lmChatGoogleGemini",
  "typeVersion": 1,
  "parameters": {{}}
}}
```

**Simple Memory**
```json
{{
  "name": "Simple Memory",
  "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
  "typeVersion": 1.3,
  "parameters": {{}}
}}
```

**Gmail: Send a message**
```json
{{
  "name": "Send a message",
  "type": "n8n-nodes-base.gmail",
  "typeVersion": 2.1,
  "parameters": {{
    "sendTo": "you@example.com",
    "subject": "Chat History",
    "emailType": "text",
    "message": "=input  : {{ '{{' }} $('When chat message received').item.json.chatInput {{ '}}' }}\noutput : {{ '{{' }} $json.output {{ '}}' }}",
    "options": {{
      "appendAttribution": false
    }}
  }}
}}
```

---

## Appendix B — Example payloads

**Minimal JSON body**
```json
{{ "chatInput": "Hello from prod" }}
```

**Extended JSON body**
```json
{{
  "chatInput": "Order status 12345",
  "metadata": {{
    "channel": "web",
    "userId": "abc-123"
  }}
}}
```

---


