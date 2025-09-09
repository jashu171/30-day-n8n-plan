# Webhooks — Making n8n Listen to the World (with Postman)

A clear, beginner-friendly guide to receive **POST** requests in n8n via **Webhook**, validate input, and return a JSON response. Includes a step-by-step Postman walkthrough and an importable n8n workflow JSON.

---

## Goal

Make n8n receive HTTP POST requests:
- Accept a JSON payload like `{"name":"Asha","age":22}`
- Decide with **If**: age ≥ 18?
- Return a clear JSON response.

---

## Prerequisites

- n8n is running (default local URL: `http://localhost:5678`)
- You are in the n8n **Editor** (canvas)
- Internet access (for ngrok/public testing)
- Postman (Web or Desktop)

---

## Step-by-Step in n8n (with sub-steps)

### 1) Add Webhook (entry point)
- In the editor, click **+** → search **Webhook** → add.
- Configure:
  - **HTTP Method:** `POST`
  - **Path:** `listen/basic`
  - **Response Mode:** `responseNode`  (we will send the reply via **Respond to Webhook**)

**Why:** n8n creates two URLs automatically:
- **Test URL** — works only while you click **Execute workflow**
- **Production URL** — works when the workflow is **Active**

---

### 2) Add If (check the payload)
- Click **+** → add **If** → connect **Webhook → If**.
- Configure **Number** condition:
  - **Value 1:** `={{ $json["age"] }}`
  - **Operation:** `largerEqual`
  - **Value 2:** `={{ 18 }}`

Meaning: If `age >= 18` → **True**, else → **False**.

---

### 3) Add Set nodes for clear messages
- **True branch** (from If’s green output): add **Set** with:
  - Keep Only Set: **ON**
  - Fields:
    - string `message` = `Adult — access permitted`
    - string `received_name` = `={{ $json["name"] }}`
    - number `age` = `={{ $json["age"] }}`
- **False branch** (from If’s red output): add **Set** with:
  - Keep Only Set: **ON**
  - Fields:
    - string `message` = `Minor — access limited`
    - string `received_name` = `={{ $json["name"] }}`
    - number `age` = `={{ $json["age"] }}`

---

### 4) Add Respond to Webhook (send the reply)
- Add **Respond to Webhook** node.
- Connect **Set (True) → Respond** and **Set (False) → Respond**.
- Configure:
  - **Response Code:** `200`
  - **Response Body:** `={{ $json }}`

Why: Returns the JSON you built in the Set node.

---

## Visual Flow (ASCII)

```
Client (POST JSON)
        |
   [Webhook /listen/basic]
        |
         [If]  (age >= 18 ?)
        /                \
  True (adult)       False (minor)
       |                   |
   [Set message]       [Set message]
        \               /
         [Respond to Webhook]
                 ^
              HTTP 200 JSON
```

---

## How to Test

### A) Local test (Test URL)
1. Click **Execute workflow** (top-right).
2. Copy the **Test URL** displayed in the Webhook node.
3. Send a POST:
   ```bash
   curl -X POST "<PASTE_TEST_URL_HERE>" \
     -H "Content-Type: application/json" \
     -d '{"name":"Asha","age":22}'
   ```
4. You should receive:
   ```json
   { "message":"Adult — access permitted","received_name":"Asha","age":22 }
   ```

### B) Public test with ngrok (Production URL)
1. Run ngrok:
   ```bash
   ngrok http 5678
   ```
2. Copy the **https** URL from ngrok, for example:
   ```
   https://xyz.ngrok-free.app
   ```
3. Production Webhook URL:
   ```
   https://xyz.ngrok-free.app/webhook/listen/basic
   ```
4. In n8n, toggle the workflow **Active**.
5. Send a POST:
   ```bash
   curl -X POST "https://xyz.ngrok-free.app/webhook/listen/basic" \
     -H "Content-Type: application/json" \
     -d '{"name":"Ravi","age":15}'
   ```
6. You should receive:
   ```json
   { "message":"Minor — access limited","received_name":"Ravi","age":15 }
   ```

---

## C) Test with Postman (Web or Desktop)

> This section is written for first-time Postman users.

### 1) Sign in to Postman
- Open the Postman website:
  ```
  https://www.postman.com/
  ```
- Login page:
  ```
  https://identity.getpostman.com/login
  ```
- Create an account if needed. You can use **Postman Web** in the browser or install the **Desktop App**.

### 2) Create a new request
- In Postman, click **New** → **HTTP Request**.
- A new request tab opens.

### 3) Choose method and URL
- Set **Method** to **POST**.
- Paste the URL:
  - **Test URL** (works only while **Execute workflow** is active), or
  - **Production URL** (works when the n8n workflow is **Active**)
- Examples:
  ```
  <PASTE_TEST_URL_FROM_WEBHOOK_NODE>
  ```
  or
  ```
  https://xyz.ngrok-free.app/webhook/listen/basic
  ```

### 4) Set headers
- Go to the **Headers** tab.
- Add:
  - **Key:** `Content-Type`
  - **Value:** `application/json`

### 5) Set body (send your values)
- Go to the **Body** tab.
- Select **raw**.
- On the right of “raw”, choose **JSON**.
- Paste one of these payloads:
  ```json
  {
    "name": "Asha",
    "age": 22
  }
  ```
  ```json
  {
    "name": "Ravi",
    "age": 15
  }
  ```

### 6) Send and inspect the response
- Click **Send**.
- Check the **Response** panel:
  - If age ≥ 18:
    ```json
    { "message":"Adult — access permitted","received_name":"Asha","age":22 }
    ```
  - If age < 18:
    ```json
    { "message":"Minor — access limited","received_name":"Ravi","age":15 }
    ```

### Optional: Environment variable for base URL
- Click the **eye** icon (top-right) → **Add** Environment (e.g., `n8n-dev`).
- Add a variable:
  - **Variable:** `BASE_URL`
  - **Initial/Current value:** your base, for example:
    ```
    https://xyz.ngrok-free.app
    ```
- Use it in the request URL:
  ```
  {{BASE_URL}}/webhook/listen/basic
  ```
- Select the environment in the top-right dropdown, then **Send**.

---

## Copyable Payloads and curl

### JSON payloads (use in Postman Body → raw → JSON)
```json
{
  "name": "Asha",
  "age": 22
}
```

```json
{
  "name": "Ravi",
  "age": 15
}
```

### curl (CLI) equivalents
```bash
curl -X POST "<PASTE_TEST_OR_PRODUCTION_URL>" \
  -H "Content-Type: application/json" \
  -d '{"name":"Asha","age":22}'
```

```bash
curl -X POST "<PASTE_TEST_OR_PRODUCTION_URL>" \
  -H "Content-Type: application/json" \
  -d '{"name":"Ravi","age":15}'
```

---

## Tips & Common Fixes

- **“Could not activate the workflow”**  
  Another workflow already uses the same Webhook path or method. Change **Path** (e.g., `listen/basic-2`) or disable the other workflow.

- **“Webhook URL mismatch”**  
  Use **Test URL** only when **Execute workflow** is active.  
  Use **Production URL** only when the workflow is **Active**.

- **404 / 405 errors**  
  Check method (POST vs GET) and verify the exact path. Confirm the ngrok URL matches your current session.

- **Postman SSL errors**  
  Ensure you use **https** (ngrok gives an https URL). Avoid disabling SSL verification unless necessary.

- **No response in Postman**  
  For **Test URL**, make sure n8n is in **Execute workflow** state; otherwise the Test URL does not listen.  
  For **Production URL**, toggle the workflow to **Active**.

---


## Key Takeaways

- **Webhook** is the entry point for external POST requests.
- **If** + **Set** handle simple decision logic and clean responses.
- **Respond to Webhook** returns the final JSON to Postman/curl/clients.
