# If / Else Workflow — n8n

**Professional goal:** Build a simple, production-friendly n8n workflow that accepts two numeric inputs (value1 and value2), compares them, and returns a basic calculation depending on the result. This guide is written for someone who is new to n8n and provides clear, step-by-step instructions with exact values and test examples.

---

## What this workflow does (short)

1. Receive two numbers (`value1` and `value2`) via an HTTP webhook (you can use a small HTML form or `curl`).
2. Use an **If** node to check: **is `value1` > `value2`?**

   * If **yes** → compute `difference = value1 - value2` and return a message.
   * If **no**  → compute `sum = value1 + value2` and return a message.
3. Respond to the caller with a JSON result containing `message` and `result`.

**Simple diagram:**

`Webhook` → `If` → (True) `Set (True)` → `Respond to Webhook`

                                            ↘ (False) `Set (False)` → `Respond to Webhook`

---

## Prerequisites (one-time)

➜ You have access to an n8n instance (cloud, local, or Docker).

➜ You know how to open the n8n Editor and import a workflow (we include import JSON below).

➜ If testing locally and you want to hit the webhook from the public web, you can use **ngrok** (optional). If you use n8n cloud, you can use the webhook URL directly.

---

## Nodes used (short)

* **Webhook** — receives input from form / curl.
* **If** — compares the two numeric values.
* **Set (True)** — calculates `value1 - value2` and sets a message.
* **Set (False)** — calculates `value1 + value2` and sets a message.
* **Respond to Webhook** — returns a JSON response to the caller.

---

## Step-by-step setup (copy each step exactly)

> **Tip:** Follow the steps in order. Each step shows the exact value to type/select.

### 1) Add the Webhook node → receive inputs

1. In the workflow editor click **+** → search and add **Webhook**.
2. Configure the Webhook node as follows:

   * **HTTP Method:** `POST`
   * **Path:** `if-else-demo`
     (This makes the full path `/webhook/if-else-demo` on your n8n domain.)
   * **Response Mode:** `Last Node`
     (This tells n8n to wait for the Respond to Webhook node so we can return the computed result.)
3. Leave other options as default.

**→ After this step:** your Webhook node is ready to receive JSON like `{ "value1": 5, "value2": 3 }`.

### 2) Add the If node → compare the two values

1. Click **+** → add **If** node.
2. Connect **Webhook** → **If** (drag connection from the Webhook node output to the If node).
3. Configure the If node **conditions** exactly like this:

   * Click **Add Condition** → choose **Number** type.
   * For the left side (value1) use an **expression**. Enter the expression box (click the gears or the small expression icon) and paste:

```
{{$json["value1"]}}
```

* **Operation:** `larger`
* For the right side (value2) use expression:

```
{{$json["value2"]}}
```

**→ After this step:** If node returns two paths: **true** (value1 > value2) and **false** (value1 ≤ value2).

### 3) Add Set (True) node → result when value1 > value2

1. Click **+** → add **Set** node. Rename it to **Set (True)**.
2. Connect the **If** node **true** output (index 0) to **Set (True)**.
3. Configure values in the Set node exactly like:

   * Click **Add Value** → choose **Number** → **Name:** `result` → **Value (expression):**

```
={{$json["value1"] - $json["value2"]}}
```

* Click **Add Value** → choose **String** → **Name:** `message` → **Value (expression):**

```
={{"value1 is greater than value2"}}
```

**→ After this step:** Set (True) will output a JSON with `result` and `message` for the true path.

### 4) Add Set (False) node → result when value1 ≤ value2

1. Click **+** → add **Set** node. Rename it to **Set (False)**.
2. Connect the **If** node **false** output (index 1) to **Set (False)**.
3. Configure values in the Set node exactly like:

   * **Number** → **Name:** `result` → **Value (expression):**

```
={{$json["value1"] + $json["value2"]}}
```

* **String** → **Name:** `message` → **Value (expression):**

```
={{"value1 is not greater than value2"}}
```

**→ After this step:** Set (False) will output a JSON with `result` and `message` for the false path.

### 5) Add Respond to Webhook node → return the result

1. Click **+** → add **Respond to Webhook** node.
2. Connect both **Set (True)** and **Set (False)** outputs to **Respond to Webhook** (so whichever path runs will feed the Respond node).
3. Configure Respond to Webhook node:

   * **Response Mode:** `Last Node` (default when used this way; if not, set it.)
   * **Response Data:** use an **expression** to return the last JSON; set it to:

```
={{$json}}
```

**→ After this step:** the workflow will send the `result` and `message` back to whoever called the webhook.

### 6) Save, activate (or execute) and test

1. Click **Save**.
2. For testing, either:

   * Click **Execute Workflow** (manual) and then send the test POST request, **or**
   * **Activate** the workflow (needed if you want the webhook to stay live permanently).

---

## How to test (two quick ways)

### A — Test with `curl` (recommended)

Run this from a terminal (replace `<YOUR_N8N_DOMAIN>` with your n8n URL; for local testing use your ngrok URL or `http://localhost:5678` if n8n is accessible):

```bash
curl -X POST 'https://<YOUR_N8N_DOMAIN>/webhook/if-else-demo' \
  -H 'Content-Type: application/json' \
  -d '{"value1":10,"value2":3}'
```

**Expected response (value1 > value2):**

```json
{
  "message": "value1 is greater than value2",
  "result": 7
}
```

Try another test where value1 is smaller:

```bash
curl -X POST 'https://<YOUR_N8N_DOMAIN>/webhook/if-else-demo' \
  -H 'Content-Type: application/json' \
  -d '{"value1":2,"value2":5}'
```

**Expected response (value1 is not greater than value2):**

```json
{
  "message": "value1 is not greater than value2",
  "result": 7
}
```

### B — Quick HTML form (open in browser and submit)

Save this as a small `form.html` and open it in your browser (replace the action URL):

```html
<form action="https://<YOUR_N8N_DOMAIN>/webhook/if-else-demo" method="post">
  <label>Value 1: <input name="value1" type="number" required></label>
  <label>Value 2: <input name="value2" type="number" required></label>
  <button type="submit">Send</button>
</form>
```

When you submit, you should see the JSON response (depends on browser behavior). If the browser shows the raw JSON, it is the webhook response.

---

## Troubleshooting & tips

➜ **Empty or string values:** If input values come as strings (e.g., from HTML form fields), the If node might compare lexicographically. To be safe, convert to number in a small **Function** node or use `Number()` in expressions: `={{Number($json["value1"])}}`.

➜ **Webhook not reachable on localhost:** If you test from external devices, run **ngrok http 5678** and use the provided ngrok URL with `/webhook/if-else-demo` path.

➜ **JSON not returned:** Make sure **Webhook** `Response Mode` is `Last Node` so the Respond to Webhook node returns the value.

➜ **Check execution:** Use the Execution History in n8n to see node input/output for debugging.

---

## Master copy (one code block — copy everything below and paste into `README.md` in your repo)

```markdown
# If / Else Workflow — n8n

**Professional goal:** Build a simple, production-friendly n8n workflow that accepts two numeric inputs (value1 and value2), compares them, and returns a basic calculation depending on the result. This guide is written for someone who is new to n8n and provides clear, step-by-step instructions with exact values and test examples.

---

## What this workflow does (short)

1. Receive two numbers (`value1` and `value2`) via an HTTP webhook (you can use a small HTML form or `curl`).
2. Use an **If** node to check: **is `value1` > `value2`?**
   - If **yes** → compute `difference = value1 - value2` and return a message.
   - If **no**  → compute `sum = value1 + value2` and return a message.
3. Respond to the caller with a JSON result containing `message` and `result`.

**Simple diagram:**

`Webhook` → `If` → (True) `Set (True)` → `Respond to Webhook`

                                            ↘ (False) `Set (False)` → `Respond to Webhook`

---

## Prerequisites (one-time)

➜ You have access to an n8n instance (cloud, local, or Docker).

➜ You know how to open the n8n Editor and import a workflow (we include import JSON below).

➜ If testing locally and you want to hit the webhook from the public web, you can use **ngrok** (optional). If you use n8n cloud, you can use the webhook URL directly.

---

## Nodes used (short)

- **Webhook** — receives input from form / curl.
- **If** — compares the two numeric values.
- **Set (True)** — calculates `value1 - value2` and sets a message.
- **Set (False)** — calculates `value1 + value2` and sets a message.
- **Respond to Webhook** — returns a JSON response to the caller.

---

## Step-by-step setup (copy each step exactly)

> **Tip:** Follow the steps in order. Each step shows the exact value to type/select.

### 1) Add the Webhook node → receive inputs

1. In the workflow editor click **+** → search and add **Webhook**.
2. Configure the Webhook node as follows:
   - **HTTP Method:** `POST`
   - **Path:** `if-else-demo`
   - **Response Mode:** `Last Node`

### 2) Add the If node → compare the two values

1. Click **+** → add **If** node.
2. Connect **Webhook** → **If**.
3. Configure the If node conditions:

```

Left (expression): {{\$json\["value1"]}}
Operation: larger
Right (expression): {{\$json\["value2"]}}

```

### 3) Add Set (True) node → result when value1 > value2

Set values:

```

result (number) = {{\$json\["value1"] - \$json\["value2"]}}
message (string) = "value1 is greater than value2"

```

### 4) Add Set (False) node → result when value1 ≤ value2

Set values:

```

result (number) = {{\$json\["value1"] + \$json\["value2"]}}
message (string) = "value1 is not greater than value2"

````

### 5) Add Respond to Webhook node → return the result

Response Data: `={{$json}}`

### 6) Save, activate (or execute) and test

- Click **Save**.
- Click **Execute Workflow** (for manual test) or **Activate** to make the webhook permanent.

---

## How to test (curl)

```bash
curl -X POST 'https://<YOUR_N8N_DOMAIN>/webhook/if-else-demo' \
  -H 'Content-Type: application/json' \
  -d '{"value1":10,"value2":3}'
````

---

## Troubleshooting & tips

* If values arrive as strings, use `Number()` in expressions or add a small Function node to convert them.
* Use ngrok for local testing if your n8n is not public.

---

## n8n import JSON is below (copy and paste into n8n Import)

```
PLACEHOLDER_JSON
```

```

---

_End of master copy._
```

---

## n8n Importable JSON (paste this JSON into n8n -> Import)

```json
{
  "name": "If-Else Demo",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "if-else-demo",
        "responseMode": "lastNode"
      },
      "id": "1",
      "name": "Webhook",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [250, 300]
    },
    {
      "parameters": {
        "conditions": {
          "number": [
            {
              "value1": "={{$json[\"value1\"]}}",
              "operation": "larger",
              "value2": "={{$json[\"value2\"]}}"
            }
          ]
        }
      },
      "id": "2",
      "name": "If",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [450, 300]
    },
    {
      "parameters": {
        "values": {
          "number": [
            {
              "name": "result",
              "value": "={{$json[\"value1\"] - $json[\"value2\"]}}"
            }
          ],
          "string": [
            {
              "name": "message",
              "value": "={{\"value1 is greater than value2\"}}"
            }
          ]
        }
      },
      "id": "3",
      "name": "Set (True)",
      "type": "n8n-nodes-base.set",
      "typeVersion": 1,
      "position": [650, 240]
    },
    {
      "parameters": {
        "values": {
          "number": [
            {
              "name": "result",
              "value": "={{$json[\"value1\"] + $json[\"value2\"]}}"
            }
          ],
          "string": [
            {
              "name": "message",
              "value": "={{\"value1 is not greater than value2\"}}"
            }
          ]
        }
      },
      "id": "4",
      "name": "Set (False)",
      "type": "n8n-nodes-base.set",
      "typeVersion": 1,
      "position": [650, 360]
    },
    {
      "parameters": {
        "responseMode": "lastNode",
        "responseData": "={{$json}}"
      },
      "id": "5",
      "name": "Respond to Webhook",
      "type": "n8n-nodes-base.respondToWebhook",
      "typeVersion": 1,
      "position": [850, 300]
    }
  ],
  "connections": {
    "Webhook": {
      "main": [
        [
          {
            "node": "If",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "If": {
      "main": [
        [
          {
            "node": "Set (True)",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Set (False)",
            "type": "main",
            "index": 1
          }
        ]
      ]
    },
    "Set (True)": {
      "main": [
        [
          {
            "node": "Respond to Webhook",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Set (False)": {
      "main": [
        [
          {
            "node": "Respond to Webhook",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {}
}
```

---

## Final notes

* The document above contains both a clear, step-by-step human-readable guide and an importable n8n JSON. Use the JSON to import quickly into n8n (Editor → Import → Paste JSON).

* If you want the README as a downloadable `.md` file and the workflow as a downloadable `.json` file, tell me and I will provide them separately.

---

*End of file.*
