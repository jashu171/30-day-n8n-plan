# HTTP Requests — Making n8n Talk to the World

## Concept
HTTP requests are like sending letters. For a GET request, you are asking: "Please send me information."

## Goal
Use the HTTP Request node (GET) to fetch a sample TODO item from a public API and present a tidy output.

We will build:
- Manual Trigger → HTTP Request (GET) → Set (optional formatting)

---

## Step-by-Step Guide

### 1) Add Manual Trigger
- In the n8n editor, click **+** and search for **Manual Trigger**.
- Add it to the canvas and place it at the start of your workflow.
- No configuration is required for this node.

### 2) Add HTTP Request (GET)
- Click **+** and add **HTTP Request**.
- Connect **Manual Trigger → HTTP Request**.
- Configure the HTTP Request node:
  - **HTTP Method:** `GET`
  - **URL:** `https://jsonplaceholder.typicode.com/todos/1`
  - **Response Format:** `JSON`
  - Leave headers, authentication, and other options as defaults for this demo.
- Purpose: This retrieves one demo TODO item with fields like `id`, `userId`, `title`, `completed`.

### 3) (Optional) Add Set to Format Output
- Add a **Set** node and connect **HTTP Request → Set (Format)**.
- In the Set node:
  - Turn **Keep Only Set** to **ON**.
  - Add fields:
    - **string**
      - `message` = `Fetched TODO successfully`
      - `title` = `={{ $json["title"] }}`
      - `completed` = `={{ $json["completed"] }}`
    - **number**
      - `todo_id` = `={{ $json["id"] }}`
- Purpose: Produces a compact, readable result for quick verification.

---

## Visual Flow (Mock Diagram)

```
Manual Trigger
      ⬇
HTTP Request (GET)
  URL: https://jsonplaceholder.typicode.com/todos/1
      ⬇
 Set (Format output)  ← optional
```

---

## Run and Verify

1. Click **Execute Workflow** in the editor.
2. Inspect the **HTTP Request** node:
   - Confirm it returns one item and shows the JSON payload from the remote API.
3. If you added **Set (Format)**, open it to see a concise output similar to:
   ```json
   {
     "message": "Fetched TODO successfully",
     "todo_id": 1,
     "title": "delectus aut autem",
     "completed": false
   }
   ```

---

## Tips and Common Fixes

- Network errors or timeouts: Re-run the workflow and verify your internet connection.
- Non-200 status codes: Inspect the HTTP Request node output to view error details from the API.
- Using a different API: Replace the URL and, if required, configure **Query Parameters**, **Headers**, or **Authentication** in the HTTP Request node.

---

## Key Takeaways

- GET requests retrieve data and generally do not require a request body.
- A minimal workflow for GET: Manual Trigger → HTTP Request.
- Add a Set node to present readable outputs for demos and testing.
