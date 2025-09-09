# HTTP Requests — Making n8n Talk to the World (GET Only)

Audience: This guide is written for first‑time n8n users and non‑technical readers. Every action is described with explicit steps and sub‑steps.

---

## What You Will Build
A minimal workflow that fetches data from a public API using a GET request and (optionally) formats the response for easy reading.

Workflow:
```
Manual Trigger → HTTP Request (GET) → Set (optional)
```
Expected result (example):
```json
{
  "message": "Fetched TODO successfully",
  "todo_id": 1,
  "title": "delectus aut autem",
  "completed": false
}
```

---

## Step 1 — Create a New Workflow  (drag and drop required)
1. On the top bar, click **New** to create a blank workflow.
2. Click the workflow name (top‑left) and rename it to: **HTTP GET — Basic Demo**.
3. Click **Save** (or ensure Auto‑save is enabled).

Checklist:
- You should see an empty canvas.
- The left panel shows the node list and search.

---

## Step 2 — Add the Manual Trigger Node  (drag and drop required)
1. In the left node panel, click the search bar and type: **Manual Trigger**.
2. Drag **Manual Trigger** from the panel and drop it near the left side of the canvas.
3. Click the node title and rename it to: **Manual Trigger** (optional).
4. Configuration: No settings are required for this node.

Verify:
- The Manual Trigger node is visible on the canvas.
- No warnings or errors are shown on the node.

---

## Step 3 — Add the HTTP Request Node (GET)  (drag and drop + connection required)
A. Place the node
1. In the left node panel, search for **HTTP Request**.
2. Drag **HTTP Request** to the canvas and drop it to the right of the Manual Trigger node.
3. Create a connection:
   - Click the small handle (dot) on the right side of **Manual Trigger**.
   - Drag the line to the left side handle of **HTTP Request** until it snaps.
   - Release the mouse to connect.

B. Configure the node
1. Click the **HTTP Request** node to open the configuration panel on the right.
2. Fill out these key fields:
   - **HTTP Method:** `GET`
   - **URL:** `https://jsonplaceholder.typicode.com/todos/1`
   - **Response Format:** `JSON` (keep default)
   - Leave **Headers**, **Authentication**, and **Query Parameters** empty for this simple demo.
3. Optional advanced settings (only if you need them):
   - **Options → Timeout**: Increase if the external API is slow.
   - **Ignore SSL Issues**: Keep off unless you know you need it.

C. Test the node alone (optional but recommended)
1. With the **HTTP Request** node selected, click **Execute Node** in the right panel.
2. In the bottom panel (or node output), open the **JSON** tab and confirm you see fields like `userId`, `id`, `title`, `completed`.

Troubleshooting:
- If you see a non‑200 status code, look for the error details in the node output.
- If no output appears, check your internet connection or try again.

---

## Step 4 — Optional: Add a Set Node to Format Output  (drag and drop + connection required)
A. Place the node
1. In the left panel, search for **Set**.
2. Drag **Set** to the canvas and drop it to the right of **HTTP Request**.
3. Connect **HTTP Request → Set** using the same drag‑from‑dot method.

B. Configure the node
1. Click the **Set** node.
2. Turn **Keep Only Set** to **ON** (this hides the unused parts of the response).
3. Add fields:
   - **string**
     - Name: `message`  | Value: `Fetched TODO successfully`
     - Name: `title`    | Value: `={{ $json["title"] }}`
     - Name: `completed`| Value: `={{ $json["completed"] }}`
   - **number**
     - Name: `todo_id`  | Value: `={{ $json["id"] }}`

C. Verify formatting
1. Click **Execute Node** on the Set node.
2. The output should show only the four fields you set.

Notes:
- The double curly braces `{{ ... }}` means “use an expression” to read a value from the previous node’s JSON.

---

## Step 5 — Run the Entire Workflow
1. Click **Execute Workflow** (top‑right).
2. The nodes should run in order: Manual Trigger → HTTP Request → Set (optional).
3. Open the final node (Set if present, otherwise HTTP Request) and review the output.

If successful, you should see data similar to:
```json
{
  "message": "Fetched TODO successfully",
  "todo_id": 1,
  "title": "delectus aut autem",
  "completed": false
}
```

---

## Step 6 — Variant: Using Query Parameters (Optional)
You can pass query parameters without editing the URL string directly.

Example API to test: `https://httpbin.org/get`

1. Duplicate your **HTTP Request** node (right‑click → **Duplicate**) or add a new one.
2. Configure:
   - **HTTP Method:** `GET`
   - **URL:** `https://httpbin.org/get`
3. Open **Query Parameters** section:
   - Click **Add Parameter** and enter:
     - Name: `userId`
     - Value: `1`
4. Execute the node and open the JSON output.
   - You should see your query parameter echoed in `args` inside the response.

---

## Visual Diagram (Text Mockup)

```
[Manual Trigger] --(connection)--> [HTTP Request (GET)] --(connection)--> [Set (Format)]

Legend:
- Square brackets [] are nodes you drag onto the canvas.
- Arrows show the connections you create by dragging from the dot on one node to the next.
```

---

## Common Errors and Fixes
1) HTTP 401 / 403 (Unauthorized / Forbidden)  
   - The demo URL does not require auth. If your own API does, configure **Authentication** in the HTTP Request node.

2) HTTP 404 (Not Found)  
   - Check the URL spelling and path.

3) HTTP 429 (Too Many Requests)  
   - The public API rate limit was exceeded. Wait and try again.

4) Empty or malformed JSON  
   - Ensure **Response Format** is set to JSON if the endpoint returns JSON.

5) No internet or blocked by firewall  
   - Verify network access from the machine running n8n.

---

## Summary
- Manual Trigger starts the flow.
- HTTP Request (GET) retrieves data from a URL.
- Set (optional) formats and simplifies the output.
- Use Query Parameters in the node when an API expects them.
