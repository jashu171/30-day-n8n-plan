# n8n AI Chat Bot Workflow — Ngrok Tunnel Guide (Dev-Only)

This guide shows how to expose your **local n8n** instance through **ngrok** so the **“When chat message received”** trigger can receive messages from the public internet. This is intended **only for development/testing**.

> **Bad practice for production:** Pointing a production URL to a developer laptop via ngrok is **not recommended**. See **Why this is bad practice** and **Safer alternatives** at the end.

---

## What you will build
A simple chat workflow that receives a message on a public URL (via ngrok), runs it through an **AI Agent** backed by **Google Gemini** with **Simple Memory**, and emails the transcript via **Gmail**.

```
Public (ngrok URL) ──▶ When chat message received ──▶ AI Agent ──▶ Gmail: Send a message
                                         ▲                 ▲
                     Google Gemini Chat Model ─────────────┘
                     Simple Memory ────────────────────────┘
```

Screenshots live in `./images`:
- `./images/ngrok-terminal.png` — ngrok running and showing a public URL
- `./images/n8n-workflow.png` — nodes wired up in n8n
- `./images/chat-ui.png` — sample chat page

---

## Prerequisites
- A **local n8n** instance running at `http://localhost:5678` (Docker or npx are fine).
- **ngrok** installed and an **authtoken** (free account works for quick tests).
- **Google Gemini** (PaLM/Gemini) API credential in n8n.
- **Gmail OAuth2** credential in n8n with permission to send email to yourself for testing.

> If you already have the workflow built from the local guide, you can skip to **4) Test the public webhook** below after starting ngrok.

---

## 1) Install and log in to ngrok
**Copy-paste:**

```bash
# Mac (Homebrew)
brew install ngrok/ngrok/ngrok

# Linux (Debian/Ubuntu)
curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null
echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list
sudo apt update && sudo apt install ngrok

# Windows (Scoop)
scoop install ngrok
```

Get your **authtoken** from the ngrok dashboard and register it:

```bash
ngrok config add-authtoken <YOUR_AUTHTOKEN>
```

---

## 2) Start a tunnel to n8n
Start ngrok so it forwards the internet to your local n8n on port 5678:

```bash
ngrok http http://localhost:5678
```

ngrok will print an **HTTPS forwarding URL**, e.g.:

```
https://example-subdomain.ngrok.app  ->  http://localhost:5678
```

Keep this terminal open while you test.

> Tip: If you need a stable subdomain, use a **reserved domain** (paid plan) and run:
> ```bash
> ngrok http --domain=<YOUR-RESERVED>.ngrok.app http://localhost:5678
> ```

---

## 3) Point the chat trigger to the ngrok URL
Open your n8n workflow and the **When chat message received** node.

- Ensure the workflow is **Active** (webhook/trigger nodes must be active to receive traffic).
- In the node, copy the **Public URL** (it usually looks like `http://localhost:5678/webhook/<id>`).
- **Replace only the scheme+host** with your ngrok URL, keeping the path as-is.

**Example:**

Original public URL (from n8n):
```
http://localhost:5678/webhook/d400435f-44fd-4587-8cff-f74848b47cb2
```

Use this over ngrok:
```
https://example-subdomain.ngrok.app/webhook/d400435f-44fd-4587-8cff-f74848b47cb2
```

You can now send requests from anywhere to the ngrok URL and they will hit your local node.

---

## 4) Test the public webhook
Use curl with your **ngrok URL** + the webhook path from the trigger:

```bash
curl -X POST "https://example-subdomain.ngrok.app/webhook/d400435f-44fd-4587-8cff-f74848b47cb2"   -H "Content-Type: application/json"   -d '{"chatInput":"Hello from ngrok on 2025-10-09"}'
```

If the workflow is wired like the example, you should receive a **Gmail** message containing both the input and the **AI Agent** output.


---

## Troubleshooting
- **404/Not Found:** Confirm the workflow is **Active** and you copied the correct path from the trigger.
- **401/403:** Check node auth options (if configured) and that you are POSTing to the right URL.
- **ngrok URL changes every run:** Use a **reserved domain** or keep the same process running. If the URL changes, update any external integrations.
- **Mixed HTTP/HTTPS:** Always use the **HTTPS** ngrok URL when calling from the internet.
- **Rate limits:** Free ngrok tiers have limits. For heavy testing, consider paid tiers or a temporary cloud instance.


---

## Why this is bad practice for production
- **Availability:** Laptops sleep, reboot, and change networks. Your “prod” URL can break without warning.
- **Security:** Developer machines are not hardened internet-facing servers. Risk of credential leakage and weak TLS posture.
- **Separation of concerns:** Dev/testing incidents can impact real users if the same endpoint is used.
- **Observability & SLAs:** No proper uptime, scaling, or logging guarantees.


## Safer alternatives
- Deploy n8n to a **cloud VM** or **container platform** (Docker+reverse proxy) with a proper domain and TLS.
- Use **separate dev/staging/prod** environments and credentials.
- Put n8n behind an **API gateway** or WAF and enforce auth (keys, OAuth/JWT, IP allowlists).
- Store secrets via environment variables or a secrets manager; avoid hard-coding them in exported JSON.


---

## Appendix A — Minimal node JSON snippets

**When chat message received**
```json
{
  "name": "When chat message received",
  "type": "@n8n/n8n-nodes-langchain.chatTrigger",
  "typeVersion": 1.3,
  "parameters": {
    "public": true,
    "options": {}
  }
}
```

**AI Agent**
```json
{
  "name": "AI Agent",
  "type": "@n8n/n8n-nodes-langchain.agent",
  "typeVersion": 2.2,
  "parameters": {
    "options": {}
  }
}
```

**Google Gemini Chat Model**
```json
{
  "name": "Google Gemini Chat Model",
  "type": "@n8n/n8n-nodes-langchain.lmChatGoogleGemini",
  "typeVersion": 1,
  "parameters": {}
}
```

**Simple Memory**
```json
{
  "name": "Simple Memory",
  "type": "@n8n/n8n-nodes-langchain.memoryBufferWindow",
  "typeVersion": 1.3,
  "parameters": {}
}
```

**Gmail: Send a message**
```json
{
  "name": "Send a message",
  "type": "n8n-nodes-base.gmail",
  "typeVersion": 2.1,
  "parameters": {
    "sendTo": "you@example.com",
    "subject": "Chat History",
    "emailType": "text",
    "message": "=input  : { '{' } $('When chat message received').item.json.chatInput { '}' }\noutput : { '{' } $json.output { '}' }",
    "options": { "appendAttribution": false }
  }
}
```

---

## Appendix B — Optional `ngrok.yml` (for a reserved domain)

If you have a **reserved domain** (paid), you may keep a simple config file (`ngrok.yml`) to make the command reproducible.

```yaml
version: "2"
authtoken: <YOUR_AUTHTOKEN>

tunnels:
  n8n:
    proto: http
    addr: 5678
    hostname: <YOUR-RESERVED>.ngrok.app
```

Run with:
```bash
ngrok start --all --config ./ngrok.yml
```

---

_Last updated: 2025-10-09_
