# 🚦 Basic If / Else Workflow in n8n

## 🎯 Goal
Create a simple workflow:
- Start with a **Manual Trigger**.
- Compare two numbers using an **If Node**.
- If **Value1 > Value2** → Go to **True Path**.
- Else → Go to **False Path**.

---

## 🪜 Step-by-Step Guide

### 1️⃣ Add Trigger Node
➡️ In the n8n editor, click **+**  
➡️ Search **Manual Trigger**  
➡️ Place it as the first node

---

### 2️⃣ Add If Node
➡️ Click **+** ➝ Add **If** node  
➡️ Connect **Manual Trigger ➝ If**  

⚙️ Configure condition:
- **Value 1:** `={{2}}`  
- **Operation:** `larger`  
- **Value 2:** `={{5}}`

(Here we just use fixed numbers for clarity)

---

### 3️⃣ Add True Path Node
➡️ From **If ➝ True output**, add a **Set Node**  
➡️ Inside Set Node add:
```json
{ "message": "✅ Value1 is greater" }
```

---

### 4️⃣ Add False Path Node
➡️ From **If ➝ False output**, add a **Set Node**  
➡️ Inside Set Node add:
```json
{ "message": "❌ Value2 is greater or equal" }
```

---

## 🔄 Workflow Flow (Mock Diagram)

```
Manual Trigger
      ⬇
      If (check 2 > 5 ?)
     ↙        ↘
 True ✅      False ❌
  ⬇             ⬇
 Set Node     Set Node
  (message)    (message)
```

---

## ▶️ Run the Workflow
1. Click **Execute Workflow**.  
2. See which branch runs:  
   - If condition is **true** → message = `"✅ Value1 is greater"`  
   - Else → message = `"❌ Value2 is greater or equal"`

---

## 📥 Importable JSON

```json
{
  "name": "Basic If Else (Trigger Demo)",
  "nodes": [
    {
      "parameters": {},
      "id": "1",
      "name": "Manual Trigger",
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [250, 300]
    },
    {
      "parameters": {
        "conditions": {
          "number": [
            {
              "value1": "={{2}}",
              "operation": "larger",
              "value2": "={{5}}"
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
          "string": [
            {
              "name": "message",
              "value": "✅ Value1 is greater"
            }
          ]
        }
      },
      "id": "3",
      "name": "Set (True)",
      "type": "n8n-nodes-base.set",
      "typeVersion": 1,
      "position": [650, 200]
    },
    {
      "parameters": {
        "values": {
          "string": [
            {
              "name": "message",
              "value": "❌ Value2 is greater or equal"
            }
          ]
        }
      },
      "id": "4",
      "name": "Set (False)",
      "type": "n8n-nodes-base.set",
      "typeVersion": 1,
      "position": [650, 400]
    }
  ],
  "connections": {
    "Manual Trigger": {
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

## 📝 Key Takeaway
👉 **Manual Trigger starts the workflow**  
👉 **If Node checks a condition**  
👉 **True/False branches give different outputs**  
