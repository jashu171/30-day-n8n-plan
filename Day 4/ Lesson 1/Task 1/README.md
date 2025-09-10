
---

## 🎯 **Goal**

Use values from the **Set Node** (`status`, `score`, `fullName`) as query parameters in an **HTTP Request** to a free test API.

---

## 🪜 **Step-by-Step Demo – Set → HTTP Request**

### Step 1: Start with the Set Node

From yesterday’s setup, you already have:

```json
{
  "status": "active",
  "score": 85,
  "fullName": "John Doe"
}
```

### Step 2: Add an HTTP Request Node

* Drag in an **HTTP Request Node**.
* Configure it like this:

  * **Method**: GET
  * **URL**: `https://httpbin.org/get`
  * **Query Parameters**:

    * `status` → `={{$json["status"]}}`
    * `score` → `={{$json["score"]}}`
    * `fullName` → `={{$json["fullName"]}}`

### Step 3: Run the Workflow

* Click **Execute Workflow**.
* In the HTTP node output, you’ll see your parameters echoed back inside the `"args"` section:

```json
{
  "args": {
    "status": "active",
    "score": "85",
    "fullName": "John Doe"
  },
  "headers": { ... },
  "url": "https://httpbin.org/get?status=active&score=85&fullName=John%20Doe"
}
```

### Step 4: Show Dynamic Data Flow

* Change the values in the Set Node (`score = 42`, `fullName = Jane Doe`).
* Re-run → show how the HTTP Request automatically uses new values.

---

## 📝 **Key Takeaway for Students**

👉 **Set Node builds the data.**
👉 **HTTP Request sends the data.**
👉 Together, they form the backbone of real integrations.

---
