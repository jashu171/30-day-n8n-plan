# HTTP Requests — Sending Data with POST (n8n)

Audience: First‑time n8n users and non‑technical readers. Each step is split into clear sub‑steps with `+` and `→` indicators. Drag‑and‑drop requirements are explicitly noted.

---

## Concept

HTTP requests are like sending letters. For a POST request, you are saying: "Here's some data to save."

---

## Goal

Build a simple workflow that sends a JSON payload to a public API using **POST**, and reads the response.

Workflow overview:
```
Manual Trigger → Set (Build Payload) → HTTP Request (POST) → Set (Format Response) [optional]
```

Expected response (example from JSONPlaceholder):
```json
{
  "name": "Asha",
  "note": "Here's some data to save",
  "priority": "high",
  "category": "onboarding",
  "id": 101
}
```

---

## Prerequisites

```text
+ n8n is running (default local URL: http://localhost:5678)
+ You can open the n8n Editor (canvas view)
+ Internet access is available
```

Key terms:
```text
+ Node  → a block that performs one task (trigger, set, request, format)
+ Canvas → the whiteboard area where you place and connect nodes
+ Connection → the line from one node’s output to another node’s input
```

---

## STEP 1 — Create a New Workflow  (drag‑and‑drop: not required)

### Do
```text
+ Open the n8n Editor
  → Click “New” to start a blank workflow
  → Click the Untitled name at top‑left and rename to: HTTP POST — Basic Demo
  → Click Save (or ensure Auto‑save is enabled)
```

### Verify
```text
+ You see an empty canvas
+ The left panel shows the node search and list
```

---

## STEP 2 — Add the Manual Trigger Node  (drag‑and‑drop: required)

### Do
```text
+ In the left panel search, type: Manual Trigger
  → Drag Manual Trigger onto the canvas
  → Place it near the left edge (start of the flow)
```

### Configure
```text
+ No configuration needed (defaults are fine)
```

### Verify
```text
+ Manual Trigger node appears without warnings
```

---

## STEP 3 — Add a Set Node (Build Payload)  (drag‑and‑drop + connection: required)

### Do
```text
+ In the left panel search, type: Set
  → Drag Set onto the canvas
  → Drop it to the right of Manual Trigger
  → Create a connection:
    + Click the right dot of Manual Trigger
    + Drag the arrow to the left dot of Set until it snaps
```

### Configure (Node configuration — key fields)
```text
+ Click the Set node
  → Keep Only Set : ON
  → Add fields:
    + string → name     = Asha
    + string → note     = Here's some data to save
    + string → priority = high
    + string → category = onboarding
```

### Verify
```text
+ Select the Set node → Click Execute Node
  → Output shows only: name, note, priority, category
```

---

## STEP 4 — Add HTTP Request (POST)  (drag‑and‑drop + connection: required)

### Do
```text
+ In the left panel search, type: HTTP Request
  → Drag HTTP Request onto the canvas
  → Drop it to the right of Set
  → Connect Set → HTTP Request using the arrow drag method
```

### Configure (Node configuration — key fields)
```text
+ Click the HTTP Request node
  → Request Method    : POST
  → URL               : https://jsonplaceholder.typicode.com/posts
  → Response Format   : JSON
  → Send              : JSON (enable “JSON/Raw Parameters” mode)
  → Body (JSON)       : ={{ $json }}
    (Explanation: this passes the entire JSON from the previous Set node as the POST body)
  → Authentication    : (leave empty for this demo)
  → Headers           : (not required; Content‑Type is set automatically when sending JSON)
```

Optional settings (use only if needed):
```text
+ Options → Timeout        → increase if API is slow
+ Options → Ignore SSL     → keep OFF unless required
```

### Test this node alone (recommended)
```text
+ Select the HTTP Request node
  → Click Execute Node
  → Open the JSON output tab
  → Confirm the API response echoes your data and includes an id (e.g., 101)
```

Troubleshooting:
```text
+ Non‑200/201 code? → Read the response body for details
+ No output?        → Re‑run; confirm internet connectivity
```

---

## STEP 5 — Add a Set Node (Format Response) [optional]  (drag‑and‑drop + connection: required)

### Do
```text
+ In the left panel search, type: Set
  → Drag Set onto the canvas
  → Drop it to the right of HTTP Request
  → Connect HTTP Request → Set (Format Response)
```

### Configure (Node configuration — key fields)
```text
+ Click the Set node
  → Keep Only Set : ON
  → Add fields:
    + string → message        = Saved successfully (simulated)
    + string → echoed_name    = ={{ $json["name"] }}
    + string → echoed_note    = ={{ $json["note"] }}
    + number → post_id        = ={{ $json["id"] }}
```

### Verify
```text
+ Select the Set (Format Response) node → Click Execute Node
  → Output shows: message, echoed_name, echoed_note, post_id
```

---

## STEP 6 — Run the Entire Workflow

### Do
```text
+ Click Execute Workflow (top‑right)
  → Observe execution:
    Manual Trigger → Set (Build Payload) → HTTP Request (POST) → Set (Format Response)
```

### Verify
```text
+ If the format node is present, final output shows:
  {
    "message": "Saved successfully (simulated)",
    "echoed_name": "Asha",
    "echoed_note": "Here's some data to save",
    "post_id": 101
  }
+ If the format node is not present, check the HTTP Request node output directly
```

---

## Visual Layout (text mock diagram)

```
[Manual Trigger] → [Set (Build Payload)] → [HTTP Request (POST)] → [Set (Format Response)]
```

Legend:
```text
+ Square brackets [] are nodes
+ The arrow shows the connection you create by dragging from one node’s dot to the next
```

---

## Common Errors and Fixes

```text
+ 401 / 403 Unauthorized / Forbidden
  → Not expected for JSONPlaceholder; for your own API, configure Authentication/Headers

+ 404 Not Found
  → Check the URL

+ 415 Unsupported Media Type
  → Ensure “Send: JSON” is enabled or set Content‑Type: application/json

+ 429 Too Many Requests
  → Public API rate limit; wait and retry

+ Empty or malformed JSON
  → Ensure Response Format is JSON and the endpoint returns JSON
```

---

## Summary

```text
+ Manual Trigger starts the run
+ Set builds a clean JSON payload
+ HTTP Request (POST) sends it to the API
+ Optional Set formats the response into a tidy summary
```
