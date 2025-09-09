# 🌐 Webhooks — Making n8n Listen to the World

---

## 🎯 Goal

Make n8n receive HTTP requests from the outside world using a **Webhook**:
- Accept a JSON payload `{ "name": "Asha", "age": 22 }`
- Decide with **If**: age ≥ 18?
- Reply with a friendly JSON message.

---

## 🪜 Step-by-Step (with sub-steps)

### 1️⃣ Add Webhook (entry point)
- In the editor, click **+** → search **Webhook** → add.
- Configure:
  - **HTTP Method:** `POST`
  - **Path:** `listen/basic`
  - **Response:** leave default (we will send a reply using a Respond node)

**Why:** This creates two URLs automatically:
- **Test URL** (works only when clicking “Execute workflow”)
- **Production URL** (works when the workflow is **Active**)

---

### 2️⃣ Add If (check the payload)
- Click **+** → add **If** → connect **Webhook → If**.
- Configure **Number** condition:
  - **Value 1:** `={{ $json["age"] }}`
  - **Operation:** `largerEqual`
  - **Value 2:** `={{ 18 }}`

**Meaning:** If `age >= 18` → **True**, else → **False**.

---

### 3️⃣ Add Set nodes for clear messages (optional but helpful)
- **True branch** (from If’s green output):
  - Add **Set** → field:
    ```json
    { "message": "✅ Adult — access permitted", "received_name": "={{$json["name"]}}", "age": "={{$json["age"]}}" }
    ```
- **False branch** (from If’s red output):
  - Add **Set** → field:
    ```json
    { "message": "🟡 Minor — access limited", "received_name": "={{$json["name"]}}", "age": "={{$json["age"]}}" }
    ```

---

### 4️⃣ Add Respond to Webhook (send the reply)
- Add **Respond to Webhook** node.
- Connect **Set (True) → Respond** and **Set (False) → Respond**.
- Configure:
  - **Response Mode:** `Last node`
  - **Response Body:** `={{ $json }}`

**Why:** Returns the message you built in the Set node.

---

## 🔄 Visual Flow (mock diagram)

```
Client (POST JSON)
        ⬇
   [Webhook /listen/basic]
        ⬇
         [If]  (age >= 18 ?)
        ↙                ↘
   True (adult)       False (minor)
       ⬇                   ⬇
   [Set message]       [Set message]
        ↘               ↙
        [Respond to Webhook]
               ⬆
            HTTP 200 JSON
```

---

## ▶️ How to test (Local & Internet)

### A) Local test (Test URL)
1. Click **Execute Workflow** (top-right).
2. Copy the **Test URL** shown in the Webhook node.
3. Send a POST:
   ```bash
   curl -X POST "<PASTE_TEST_URL_HERE>"      -H "Content-Type: application/json"      -d '{"name":"Asha","age":22}'
   ```
4. You should receive:
   ```json
   { "message":"✅ Adult — access permitted","received_name":"Asha","age":22 }
   ```

### B) Public test with ngrok (Production URL)
1. Run ngrok:
   ```bash
   ngrok http 5678
   ```
2. Copy the **https** URL from ngrok, e.g. `https://xyz.ngrok-free.app`.
3. In the Webhook node, note the **Production URL** pattern:
   ```
   https://<your-host>/webhook/listen/basic
   ```
   Example:
   ```
   https://xyz.ngrok-free.app/webhook/listen/basic
   ```
4. **Activate** the workflow (toggle ON in editor).
5. Send a POST:
   ```bash
   curl -X POST "https://xyz.ngrok-free.app/webhook/listen/basic"      -H "Content-Type: application/json"      -d '{"name":"Ravi","age":15}'
   ```
6. You should receive:
   ```json
   { "message":"🟡 Minor — access limited","received_name":"Ravi","age":15 }
   ```

---

## 🧪 Tips & Common Fixes

- **“Could not activate the workflow”**  
  - Another workflow already uses the same Webhook path or method. Change **Path** (e.g., `listen/basic-2`) or disable the other workflow.

- **“Webhook URL mismatch”**  
  - Ensure you use **Test URL** only when clicking “Execute workflow”.  
  - Use **Production URL** only when the workflow is **Active**.

- **404 / 405 errors**  
  - Check method (POST vs GET) and exact path spelling.  
  - Confirm the ngrok URL matches your current session.

---

## ✅ Key Takeaways

- **Webhook** node = entry for external HTTP requests.  
- **If** + **Set** = simple, readable decision and message building.  
- **Respond to Webhook** = controlled, explicit HTTP response.
