# Email Connections — Automation Workflow: When Someone Fills a Form

**Flow:**  
💻 **Step 1 — Form submitted** → 📊 **Step 2 — Save to Sheet** → 📧 **Step 3 — Send email** → ✅ **Step 4 — All done!**

Audience: First-time n8n users and non-technical readers. Every action is split into clear sub-steps with `+` and `→` indicators. Drag-and-drop requirements are explicitly noted.

---

## Goal

When someone submits a form:
- Capture the submission in n8n (Webhook).
- Save the data to a Sheet (Google Sheets).
- Email the submitter a confirmation (SMTP).
- Return a success response to the browser/client.

---

## Prerequisites

```text
+ n8n running (default local editor: http://localhost:5678)
+ Internet access
+ Google account with access to a spreadsheet (or create a new one)
+ SMTP credentials (e.g., Gmail App Password or any SMTP provider)
+ Optional: ngrok for public testing (to receive form submissions from outside your machine)
```

---

## Visual Overview (Mock Diagram)

```
Browser / Form
      |
      v
[Webhook (POST) /form/submit] --> [Google Sheets: Append Row] --> [Email Send (SMTP)] --> [Respond to Webhook]
```

---

## Step 1 — Form Submitted (Webhook Trigger)  (drag-and-drop: required)

### Do
```text
+ In the left panel search, type: Webhook
  → Drag Webhook onto the canvas (leftmost)
  → Click the node to configure
```

### Configure (key fields)
```text
+ HTTP Method  : POST
+ Path         : form/submit
+ Response Mode: responseNode (we will reply with a final node)
```

**Why:** This generates two URLs:
```text
+ Test URL       → usable only while “Execute workflow” is active
+ Production URL → usable when the workflow is toggled Active
```

### Optional: Sample Form (copy and host locally or in your frontend)
Use this basic HTML to POST to your Webhook URL (replace the URL before testing):
```html
<!DOCTYPE html>
<html>
  <body>
    <form id="contact" method="POST" action="PASTE_YOUR_WEBHOOK_URL_HERE">
      <label>Name: <input name="name" required></label><br>
      <label>Email: <input name="email" type="email" required></label><br>
      <label>Message: <textarea name="message" required></textarea></label><br>
      <button type="submit">Send</button>
    </form>
    <script>
      // If your Webhook expects JSON, use fetch instead:
      // document.getElementById('contact').addEventListener('submit', async (e) => {
      //   e.preventDefault();
      //   const form = new FormData(e.target);
      //   const payload = Object.fromEntries(form.entries());
      //   const res = await fetch('PASTE_YOUR_WEBHOOK_URL_HERE', {
      //     method: 'POST',
      //     headers: { 'Content-Type': 'application/json' },
      //     body: JSON.stringify(payload)
      //   });
      //   const data = await res.json();
      //   alert(JSON.stringify(data));
      // });
    </script>
  </body>
</html>
```

**Important:**  
- If you submit as regular form data (`application/x-www-form-urlencoded`), the Webhook node will still parse fields.  
- If you post JSON (recommended), set `Content-Type: application/json` and send `{"name":"...", "email":"...", "message":"..."}`.

**Quick cURL tests**
```bash
# JSON body
curl -X POST "<PASTE_TEST_OR_PRODUCTION_URL>" \
  -H "Content-Type: application/json" \
  -d '{"name":"Asha","email":"asha@example.com","message":"Hello!"}'
```

---

## Step 2 — Save to Sheet (Google Sheets Append)  (drag-and-drop + connection: required)

### Do
```text
+ Search: Google Sheets
  → Drag Google Sheets onto the canvas (right of Webhook)
  → Connect Webhook → Google Sheets
```

### Configure (Append Row)
```text
+ Operation        : Append
+ Authentication   : Add new Google credential (follow the OAuth prompts)
+ Spreadsheet ID   : (Paste from the Google Sheet URL)
+ Sheet (Tab) Name : e.g., FormResponses
+ Columns Mapping  :
  → name     = {{$json["name"]}}
  → email    = {{$json["email"]}}
  → message  = {{$json["message"]}}
  → timestamp= {{$now}}    (or use an Expression: {{new Date().toISOString()}})
```

**Tips**
```text
+ Create the sheet with headers: name | email | message | timestamp
+ Ensure the Sheet (tab) name is exact (case-sensitive)
```

**Alternative (no Google account):**  
Use a CSV with the **Spreadsheet File** node (Operation: Append). Map the same fields into a local CSV file. This is good for offline demos.

---

## Step 3 — Send Email (SMTP)  (drag-and-drop + connection: required)

### Do
```text
+ Search: Email
  → Drag Email Send onto the canvas (right of Google Sheets)
  → Connect Google Sheets → Email Send
```

### Configure (SMTP)
```text
+ Authentication : Add new SMTP credential
  → Host         : smtp.gmail.com        (example)
  → Port         : 587                   (TLS)
  → User         : your-email@example.com
  → Password     : your-app-password     (or provider password)
+ From Email     : your-email@example.com
+ To Email       : {{$json["email"]}}    (send to form submitter)
+ Subject        : Thank you, {{$json["name"]}} — we received your message
+ Text           :
  Hello {{$json["name"]}},
  We received your message:
  "{{$json["message"]}}"

  We’ll get back to you shortly.
  — Team
```

**Notes**
```text
+ For Gmail, create an App Password (if 2FA is enabled) and use it here.
+ Most providers support port 587 (TLS). Use 465 for SSL if required.
```

---

## Step 4 — All done! (Respond to Webhook)  (drag-and-drop + connection: required)

### Do
```text
+ Search: Respond to Webhook
  → Drag Respond to Webhook onto the canvas (right of Email Send)
  → Connect Email Send → Respond to Webhook
```

### Configure
```text
+ Response Code : 200
+ Response Body : ={{ $json }}
```

**What it returns (example)**
```json
{
  "status": "ok",
  "info": "Email queued/sent",
  "email_to": "asha@example.com"
}
```

> If you want a custom message, insert a **Set** node before Respond to Webhook and build your own JSON response.

---

## Test It

### Local Test (Test URL)
```text
+ Click Execute workflow in n8n
+ Submit the sample HTML form or use the cURL command
+ Watch nodes turn green; check Google Sheet; verify email inbox
```

### Public Test (Production URL with ngrok)
```bash
ngrok http 5678
```
```text
+ Copy the https URL from ngrok, e.g. https://xyz.ngrok-free.app
+ Your Production Webhook URL becomes:
  https://xyz.ngrok-free.app/webhook/form/submit
+ Toggle the workflow Active in n8n
+ Submit the public form or use Postman/cURL against the Production URL
```

---

## Troubleshooting

```text
+ Webhook not receiving:
  → For Test URL, click Execute workflow first
  → For Production URL, toggle the workflow Active

+ Google Sheets “no permission”:
  → Reconnect OAuth credential; confirm the correct Google account
  → Verify Spreadsheet ID and Sheet (tab) name are correct

+ Email not sending:
  → Check SMTP host, port, user, password (app password)
  → Confirm “From Email” is allowed by your provider
  → Check spam folder for test emails

+ 404 / 405 errors:
  → Confirm POST method and exact /form/submit path

+ ngrok mismatch:
  → Use the current https URL printed by ngrok; old sessions won’t work
```

---

## Importable n8n Workflow JSON (basic version)

> Paste into **n8n → Import → Paste JSON**. After import, finish credentials for Google Sheets and SMTP.

```json
{
  "name": "Form → Sheet → Email → Respond",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "form/submit",
        "responseMode": "responseNode",
        "options": {}
      },
      "id": "1",
      "name": "Webhook (Form Submit)",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 2,
      "position": [240, 300]
    },
    {
      "parameters": {
        "operation": "append",
        "sheetId": "",
        "range": "",
        "options": {
          "locationDefine": "define",
          "valueInputMode": "RAW",
          "keyRow": 1
        },
        "columns": {
          "string": [
            { "name": "name", "value": "={{$json[\"name\"]}}" },
            { "name": "email", "value": "={{$json[\"email\"]}}" },
            { "name": "message", "value": "={{$json[\"message\"]}}" },
            { "name": "timestamp", "value": "={{$now}}" }
          ]
        }
      },
      "id": "2",
      "name": "Google Sheets (Append Row)",
      "type": "n8n-nodes-base.googleSheets",
      "typeVersion": 4,
      "position": [520, 300],
      "credentials": {
        "googleApi": {
          "id": "__PLEASE_CONFIGURE__",
          "name": "Google Account"
        }
      }
    },
    {
      "parameters": {
        "fromEmail": "",
        "toEmail": "={{$json[\"email\"]}}",
        "subject": "Thank you, {{$json[\"name\"]}} — we received your message",
        "text": "Hello {{$json[\"name\"]}},\nWe received your message:\n\"{{$json[\"message\"]}}\"\n\nWe’ll get back to you shortly.\n— Team"
      },
      "id": "3",
      "name": "Email Send (SMTP)",
      "type": "n8n-nodes-base.emailSend",
      "typeVersion": 3,
      "position": [800, 300],
      "credentials": {
        "smtp": {
          "id": "__PLEASE_CONFIGURE__",
          "name": "SMTP Account"
        }
      }
    },
    {
      "parameters": {
        "responseCode": 200,
        "responseBody": "={{ {\"status\":\"ok\",\"info\":\"Email queued/sent\",\"email_to\": $json[\"email\"]} }}"
      },
      "id": "4",
      "name": "Respond to Webhook",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 2,
      "position": [1040, 300]
    }
  ],
  "connections": {
    "Webhook (Form Submit)": {
      "main": [
        [
          { "node": "Google Sheets (Append Row)", "type": "main", "index": 0 }
        ]
      ]
    },
    "Google Sheets (Append Row)": {
      "main": [
        [
          { "node": "Email Send (SMTP)", "type": "main", "index": 0 }
        ]
      ]
    },
    "Email Send (SMTP)": {
      "main": [
        [
          { "node": "Respond to Webhook", "type": "main", "index": 0 }
        ]
      ]
    }
  },
  "active": false,
  "settings": {}
}
```

---

## Key Takeaways

```text
+ Webhook receives form submissions
+ Google Sheets stores records
+ Email Send confirms to the user
+ Respond to Webhook returns a clean success JSON
```
