#  Nested If / Else Workflow — n8n

**Professional goal:** Build a clear, step-by-step nested conditional workflow starting from a **Manual Trigger**. This guide is short, practical, and written so a first-time n8n user can implement and test it quickly.

---

## 🎯 What the workflow does (short)
1. Start with a **Manual Trigger**.
2. Primary check: **Is `value1 > value2`?** (If Node 1)
   - If **no** → end: `Value2 is greater or equal` (False path).
   - If **yes** → do a nested check (If Node 2):
     - Is **(value1 - value2) > threshold**?
       - If **yes** → `Value1 is significantly greater` (True → High branch).
       - If **no** → `Value1 is greater but margin is small` (True → Low branch).

---

## 🪜 Step-by-step (very short & exact)

### 1️⃣ Add Manual Trigger
➡️ In the n8n editor click **+** → add **Manual Trigger**.  
➡️ Place it at the left (start of the workflow).

---

### 2️⃣ Add First If node (Primary comparison)
➡️ Add **If** node and connect: **Manual Trigger ➝ If**.  
➡️ Configure the If node (Primary check):

- **Left (Value 1):** `={{20}}`  
- **Operation:** `larger`  
- **Right (Value 2):** `={{5}}`

> These are fixed example numbers. You can edit them later to test different outcomes.

---

### 3️⃣ Add Nested If node (Margin check)
➡️ From **If → True output**, add another **If** node (call it *Nested If*).  
➡️ Connect **If (true) ➝ Nested If**.  
➡️ Configure Nested If to check the margin (difference):

- **Left (Difference):** `={{20 - 5}}`  
- **Operation:** `larger`  
- **Right (Threshold):** `={{8}}`

This checks: **(value1 - value2) > 8 ?**

---

### 4️⃣ Add result nodes (one per outcome)
➡️ From **Nested If → True**, add **Set** node:
```json
{ "message": "Value1 is significantly greater" }
```

➡️ From **Nested If → False**, add **Set** node:
```json
{ "message": "Value1 is greater but margin is small" }
```

➡️ From **If → False**, add **Set** node:
```json
{ "message": "Value2 is greater or equal" }
```

---

## 🔄 Diagram (mockdown)

```
Manual Trigger
      ⬇
   If (value1 > value2 ?)
     ↙         ↘
  False        True
  (A)          ⬇
  Set       Nested If (value1 - value2 > threshold ?)
              ↙               ↘
      True (High)           False (Low)
       Set - High            Set - Low
```

**Legend:**  
- (A) = `Value2 is greater or equal` result.  
- High = `Value1 is significantly greater`.  
- Low = `Value1 is greater but margin is small`.

---

## ▶️ How to run & test (quick)
1. Click **Execute Workflow**.  
2. The workflow runs starting at **Manual Trigger**.  
3. The executed branch will show the last node output in n8n UI:
   - With the example numbers (20 and 5) → Primary = True and Nested = True → output: `Value1 is significantly greater`.
   - If you change Primary to 10 and 5 → Primary True, Nested False → output: `Value1 is greater but margin is small`.
   - If you change Primary to 3 and 5 → Primary False → output: `Value2 is greater or equal`.

> To test different values: edit the numeric values inside the If nodes and re-run.

---

## 📥 Importable JSON (copy → n8n Editor → Import)
```json
{
  "name": "Nested If-Else Demo",
  "nodes": [
    {
      "parameters": {},
      "id": "1",
      "name": "Manual Trigger",
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [200, 300]
    },
    {
      "parameters": {
        "conditions": {
          "number": [
            {
              "value1": "={{20}}",
              "operation": "larger",
              "value2": "={{5}}"
            }
          ]
        }
      },
      "id": "2",
      "name": "If - Primary",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [420, 300]
    },
    {
      "parameters": {
        "conditions": {
          "number": [
            {
              "value1": "={{20 - 5}}",
              "operation": "larger",
              "value2": "={{8}}"
            }
          ]
        }
      },
      "id": "3",
      "name": "If - Nested (Margin)",
      "type": "n8n-nodes-base.if",
      "typeVersion": 1,
      "position": [640, 280]
    },
    {
      "parameters": {
        "values": {
          "string": [
            {
              "name": "message",
              "value": "Value1 is significantly greater"
            }
          ]
        }
      },
      "id": "4",
      "name": "Set - High Margin",
      "type": "n8n-nodes-base.set",
      "typeVersion": 1,
      "position": [860, 200]
    },
    {
      "parameters": {
        "values": {
          "string": [
            {
              "name": "message",
              "value": "Value1 is greater but margin is small"
            }
          ]
        }
      },
      "id": "5",
      "name": "Set - Low Margin",
      "type": "n8n-nodes-base.set",
      "typeVersion": 1,
      "position": [860, 360]
    },
    {
      "parameters": {
        "values": {
          "string": [
            {
              "name": "message",
              "value": "Value2 is greater or equal"
            }
          ]
        }
      },
      "id": "6",
      "name": "Set - Value2 or Equal",
      "type": "n8n-nodes-base.set",
      "typeVersion": 1,
      "position": [640, 420]
    }
  ],
  "connections": {
    "Manual Trigger": {
      "main": [
        [
          {
            "node": "If - Primary",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "If - Primary": {
      "main": [
        [
          {
            "node": "If - Nested (Margin)",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Set - Value2 or Equal",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "If - Nested (Margin)": {
      "main": [
        [
          {
            "node": "Set - High Margin",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Set - Low Margin",
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

## 📝 Final notes (professional)
- This file uses **fixed example numbers** so a beginner can follow steps without adding extra nodes.  
- When you are comfortable, replace fixed numbers with dynamic values (use a **Set** node or expressions like `{{$json["value1"]}}`) to make the workflow accept input.  
- Keep the layout tidy: place nested If to the right of primary If so the flow reads left → right.

---

*End of guide.*
