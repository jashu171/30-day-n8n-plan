# Evolution Manager (WhatsApp) — Quick Start

Use the **already hosted** Evolution Manager to get WhatsApp running in minutes.

---

## 1) Log in
Open: `https://evolution-api-v1-8-7-6big.onrender.com/manager/`  
Fill the form and click **Login**:

- **Server URL:** `https://evolution-api-v1-8-7-6big.onrender.com`  
- **API Key Global:** `21uk1a66a121uk1a66a1`

![](images/1-login.png)

---

## 2) Create an instance
Click **Instance +** (top-right).

![](images/2-create-instance.png)

---

## 3) Fill instance details → Save
- **Name:** e.g. `WhatsApp Support`  
- **Channel:** `Baileys`  
- **Token:** use the generated value (keep it)  
- **Number:** **leave blank** (Baileys links via QR)

Click **Save**.

![](images/3-new-instance.png)

---

## 4) Link your WhatsApp (QR)
On the instance dashboard, click **Get QR Code**.

![](images/4-scan-qrcode.png)

On your phone: **WhatsApp → Settings → Linked devices → Link a device**, then scan the QR.  
Wait until the instance shows **Connected**.

![](images/5-logininto-whatsapp.png)

---

## 5) Set up Webhook (to receive messages)
Go to **Events → Webhook** and configure:

- **Enabled:** ON  
- **URL:** your receiver (e.g., n8n)  
  - Example: `https://YOUR_N8N_HOST/webhook/my-whats-app`  
- **Webhook Base64:** ON (helps with media)

![](images/6-webhook-setup.png)

Scroll down:

- Enable the event **`MESSAGES_UPSERT`**
- Click **Save**

![](images/7-save-the-webhook.png)

---

## 6) Test
Send a WhatsApp message to the linked number and confirm your webhook (e.g., n8n) receives the event.

**Notes**
- If the QR expires, click **Restart** then **Get QR Code** again.
- Keep the instance **Token** secret; it’s required for API calls from clients.







# n8n WhatsApp Auto-Reply — Quick Start

This guide shows how to install the **Evolution API** community node and run the WhatsApp auto-reply workflow.

---

## Prerequisite — Install the community node

1. **Go to Settings**  
   In n8n, click your avatar (bottom-left) → **Settings**.  
   ![](images/1-goto-settings.png)

2. **Open Community nodes**  
   In the left sidebar, click **Community nodes** and then click **Install** (top-right).  
   ![](images/2-community-nodes-page.png)

3. **Install the package**  
   In the modal, paste the npm package name: **`n8n-nodes-evolution-api-english`**.  
   Tick **I understand the risks…**, then click **Install**.  
   ![](images/3-install-modal.png)

4. **Confirm it shows as installed**  
   You should now see the package listed as installed.  
   ![](images/4-installed-list.png)

5. **Verify in editor**  
   Back in the workflow editor, click **+** and search **Evolution API** — the node should appear.  
   ![](images/5-find-evolution-node.png)

---

**Connections:** Webhook → AI Agent → Send text; Groq Chat Model → AI Agent (ai_languageModel); Simple Memory → AI Agent (ai_memory).

---

## Credentials

- **Groq API:** add your key.  
- **Evolution API:**  
  - Server URL: `https://evolution-api-v1-8-7-6big.onrender.com`  
  - API Key: `21uk1a66a121uk1a66a1`

---

## Point Evolution Manager to n8n

Evolution Manager → **Events → Webhook**:  
- **Enabled:** ON  
- **URL:** Production URL from n8n Webhook node  
- **Webhook Base64:** ON  
- Enable **`MESSAGES_UPSERT`** → **Save**.

---

## Test

Send a WhatsApp message to the connected number and watch an execution in n8n; the **Send text** node replies back.

