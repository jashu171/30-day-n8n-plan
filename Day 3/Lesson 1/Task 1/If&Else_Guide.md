# IF / ELSE Workflow in n8n Automation

This guide explains how to create an **IF/Else workflow** in **n8n**.  
By the end of this tutorial, you will be able to automatically make decisions in your workflows based on conditions you define (for example: send an email if a value is true, or log data if false).

---

## 🎯 Goal
- Learn how to create an **IF/Else workflow** in n8n.  
- Understand how to connect nodes step by step.  
- Build an automation that takes different actions depending on the condition.

---

## 🛠 Step-by-Step Workflow

### 1. Start Workflow
➡️ Open **n8n Editor UI**.  
➡️ Click on **“+ New Workflow”**.  
➡️ Give your workflow a **name** (e.g., `IF-Else Demo`).

---

### 2. Add a Trigger Node
➡️ Add a **Trigger Node** to start the workflow.  
For example:  
- **Manual Trigger** → good for testing.  
- **Webhook Trigger** → useful if you want external apps to call this workflow.

---

### 3. Add an IF Node
➡️ Click **“+ Add Node”**.  
➡️ Search for **IF** and select it.  
➡️ Connect the **Trigger Node → IF Node**.  

---

### 4. Configure the IF Node
➡️ In the IF node settings:  
- Choose a **Condition** (example: `Value1 > Value2`).  
- Example: Check if `Amount` is greater than `100`.  

Result:  
- If **True** → workflow will continue through **“true” branch**.  
- If **False** → workflow will continue through **“false” branch**.

---

### 5. Add Action for TRUE Branch
➡️ From the **IF node (true output)** → Add a new node.  
➡️ Example Action: **Send Email**.  
➡️ Configure your email details.  

---

### 6. Add Action for FALSE Branch
➡️ From the **IF node (false output)** → Add a new node.  
➡️ Example Action: **Write Data to Google Sheet**.  
➡️ Configure your Google Sheets integration.  

---

### 7. Test the Workflow
➡️ Click **“Execute Workflow”**.  
➡️ Provide sample input.  
➡️ Check results:  
- If condition is true → Email will be sent.  
- If condition is false → Data will be logged in Google Sheet.

---

### 8. Save & Activate
➡️ Click **“Save”** to store your workflow.  
➡️ Turn **“Active”** ON to make it live.  

---

## 📊 Visual Flow

```
[Trigger Node] 
   ⬇️
     [IF Node]
       ↘️ (True)  → [Send Email]
       ↙️ (False) → [Google Sheet]
```

---

## 📌 Best Practices
- Clearly name each node (e.g., `Check Amount`, `Send Alert Email`).  
- Test with sample data before activating.  
- Keep workflows modular and easy to maintain.

---

## 🎉 You Did It!
You have successfully created an **IF/Else Workflow** in n8n. This structure helps you automate decision-making, saving time and reducing manual effort.
