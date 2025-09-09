# IF / ELSE Workflow in n8n — Simple Step-by-Step Guide

**Reference:** base structure adapted from the user's attached README. fileciteturn0file0

---

## 🎯 Goal
Build a clear, minimal IF / ELSE workflow in n8n that checks a condition and runs different actions for **true** and **false** paths. This guide is written to be followed by non-technical users step-by-step.

---

## ✅ Prerequisites
- n8n running (local or cloud).
- Access to the n8n Editor (e.g., `http://localhost:5678`).
- Basic credential access if you plan to send email or write to Google Sheets (optional for testing).

---

## 🧭 Quick Example Summary
We will use a **Set Node** to create test data, then an **IF Node** to check a condition (`amount > 100`).  
- **True branch** → Send an email (example).  
- **False branch** → Append the data to a Google Sheet (example).

---

## 🪜 Step-by-Step (very simple)

### Step 1 — Create a New Workflow
➡️ Open **n8n Editor** → Click **+ New Workflow** → Name it `IF-ELSE Demo`.

### Step 2 — Add a Set Node (create test data)
➡️ Add **Set** node → Open node → Add fields exactly as below:

```json
{
  "status": "active",
  "score": 85,
  "fullName": "John Doe",
  "amount": 120
}
```

➡️ Save the node and connect it as the starting node.

### Step 3 — Add an IF Node and connect
➡️ Add **IF** node → Connect **Set → IF**.  
➡️ Configure IF node like this:
- **Mode**: Single (default)  
- **Left value**: `{{$json["amount"]}}`  
- **Operation**: `>` (greater than)  
- **Right value**: `100`

Result: IF node now has two outputs: **true** and **false**.

### Step 4 — Add Action for TRUE branch
➡️ From IF node **true output** → Add a **Send Email** node (or any action).  
➡️ Configure: `To`, `Subject`, `Body`. Use expressions to include data, e.g.:  
- Subject: `High amount alert for {{$json["fullName"]}}`  
- Body: `Amount received: {{$json["amount"]}}`

### Step 5 — Add Action for FALSE branch
➡️ From IF node **false output** → Add a **Google Sheets** (Append) node (or another action to log data).  
➡️ Map fields like `fullName`, `amount`, `status`, `score` into the sheet columns.

### Step 6 — Test the workflow
➡️ Click **Execute Workflow** (Play button).  
➡️ Confirm behavior:
- If `amount` > 100 → Email node runs (check Email node output).  
- If `amount` ≤ 100 → Google Sheets node runs (check sheet or node output).

### Step 7 — Save & Activate
➡️ Click **Save** → Switch **Active** ON to run automatically when trigger conditions are met.

---

## 📌 Example: Change test data
Open the **Set** node and update `amount` to `50`. Re-run to see the workflow take the **false** branch. This demonstrates how IF rules change the path dynamically.

---

## 📊 Visual Flow (ASCII)

```
[Set Node: test data]
       ⬇️
     [IF Node: amount > 100]
      ↙️       ↘️
 (false)     (true)
   ↓           ↓
[GoogleSheet] [Send Email]
```

---

## 🔍 JSON Schema (for the Set Node payload)
Use this JSON Schema to validate input before the IF node. It ensures `amount` is numeric and required fields exist.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "n8n IF-ELSE Input Schema",
  "type": "object",
  "required": ["status", "score", "fullName", "amount"],
  "properties": {
    "status": {
      "type": "string",
      "enum": ["active", "inactive", "pending"]
    },
    "score": {
      "type": "integer",
      "minimum": 0
    },
    "fullName": {
      "type": "string",
      "minLength": 1
    },
    "amount": {
      "type": "number"
    }
  },
  "additionalProperties": false
}
```

### ✅ Sample Valid JSON (passes schema)
```json
{
  "status": "active",
  "score": 85,
  "fullName": "John Doe",
  "amount": 120
}
```

### ❌ Sample Invalid JSON (fails schema)
```json
{
  "status": "unknown",
  "score": -5,
  "fullName": "",
  "amount": "one hundred"
}
```

> **How to check the schema**: use any online JSON Schema validator or a tool like **ajv** (Node.js). Example (locally): `ajv validate -s schema.json -d data.json`

---

## ✅ Quick Troubleshooting
- IF node always goes to false? → Verify the expression: `{{$json["amount"]}}` returns a number, not a string. Use `Number({{$json["amount"]}})` or ensure set node uses a numeric value.  
- Email not sent? → Check email credentials and the node execution output for errors.  
- Google Sheets append fails? → Confirm OAuth credentials and sheet ID/permissions.

---

## 📝 Final Notes
- This guide follows the concise style from your uploaded README while focusing on IF/ELSE basics. fileciteturn0file0  
- You can safely paste the **JSON Schema** into a validator to confirm your Set Node data before using it in production.

---
