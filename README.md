# AI ATS Resume Reviewer & Writer (n8n)

This project demonstrates how to build a **production‑style AI agent** using **n8n** that evaluates a resume against a job description (ATS-style), rewrites the resume, and exports the result to **Google Docs**.

> **No frontend. No backend.** Just AI + workflows + automation.

---

## What This Agent Does

1. Accepts structured inputs (resume + job description)
2. Cleans and normalizes text
3. Validates input quality
4. Evaluates the resume using an LLM (ATS-style scoring)
5. Rewrites the resume based on gaps
6. Creates a Google Doc with the tailored resume
7. Returns a structured response (score + doc link)

This is an **AI agent**, not a simple prompt.

---

## Agent Architecture (High Level)

```
Input → Clean → Validate → AI Analyze → Parse → AI Rewrite → Google Docs → Output
```

Key concepts used:

* Input validation
* Deterministic data flow
* Multiple AI roles (analysis → action)
* External tool usage (Google Docs API)
* Safe execution (no cross‑node memory assumptions)

---

## Tech Stack

* **n8n** – workflow orchestration
* **OpenAI (or compatible LLM)** – resume analysis & rewriting
* **Google Docs API** – document creation
* **JavaScript (n8n Code nodes)** – data cleaning & response shaping

---

## Prerequisites

Before running this workflow, you need:

1. An **n8n account** (cloud or self‑hosted)
2. An **LLM API key** (OpenAI / Groq / Gemini / Ollama). I used OpenAI
3. A **Google Cloud project** with:

   * Google Docs API enabled
   * Google Drive API enabled
   * OAuth consent screen configured
   * OAuth Client ID (Web Application)

---

## Google OAuth Setup (Summary)

1. Create a Google Cloud project
2. Enable:

   * Google Docs API
   * Google Drive API
3. Configure OAuth Consent Screen (External)
4. Add scopes:

   ```
   https://www.googleapis.com/auth/documents
   https://www.googleapis.com/auth/drive.file
   ```
5. Create OAuth Client ID (Web application)
6. Add redirect URI:

   ```
   https://<your-n8n-domain>/rest/oauth2-credential/callback
   ```
7. Create **Google OAuth2 API** credential in n8n and connect

---

##  Workflow Nodes (Step by Step)

### 1️⃣ Edit Fields (Input Node)

Defines all inputs in one place:

* `resume_text`
* `job_description`
* `candidate_name`
* `target_role`

> This node acts as the input layer (later replaceable by Webhook or Form).

---

### 2️⃣ Code – Clean & Normalize Text

Cleans input text to avoid LLM confusion:

* Removes invisible characters
* Normalizes bullets
* Fixes spacing
* Calculates text length

```js
const resume_text = normalize($json.resume_text);
const job_description = normalize($json.job_description);
```

---

### 3️⃣ If – Input Validation

Rules:

* Resume length ≥ 500 characters
* Job description length ≥ 300 characters

Prevents:

* Hallucinations
* Wasted tokens
* Low‑quality output

---

### 4️⃣ Message a Model – ATS Evaluation (AI Brain #1)

Purpose:

* Compare resume vs JD
* Produce structured ATS evaluation
* Identify strengths & gaps
* Output **strict JSON**

---

### 5️⃣ Code – Parse ATS JSON

Safely parses the LLM output:

* Handles malformed JSON
* Fails fast with clear errors

```js
const parsed = JSON.parse($json.message.content);
return [parsed];
```

---

### 6️⃣ Message a Model – Resume Rewriting (AI Brain #2)

Purpose:

* Rewrite resume based on gaps
* Preserve truth
* Maintain ATS‑friendly formatting

This demonstrates **agent chaining**:

> AI analyzes → AI acts

---

### 7️⃣ HTTP Request – Google Docs

Actions:

* Create a Google Doc
* Insert the rewritten resume
* Return a shareable document URL

---

### 8️⃣ Code – Final Output

Shapes the final response:

```json
{
  "status": "success",
  "ats_score": 85,
  "strengths": [...],
  "gaps": [...],
  "doc_url": "https://docs.google.com/..."
}
```

---

## ▶️ How to Run This Workflow (Correct Way)

> **Important:** For real, reliable execution, do **not** rely on Ctrl + Enter.

### ✅ Recommended: Webhook Execution

1. Add a **Webhook** node at the start
2. Method: `POST`
3. Send input JSON:

```json
{
  "resume_text": "...",
  "job_description": "...",
  "candidate_name": "Manisha",
  "target_role": "QA Engineer"
}
```

This is how **production n8n workflows run**.

---

## 🧠 Key Learnings

* AI agents ≠ single prompts
* Deterministic data flow matters
* Each node only knows what flows into it
* Editor one‑shot execution is for debugging, not production
* Real products use triggers (webhooks, schedules, forms)

---

## 🚀 Possible Extensions

* Replace Edit Fields with a Webhook
* Add a frontend (form / UI)
* Store results in a database
* Add email or Slack notifications
* Convert to a SaaS‑style API

---

## 📌 Status

✅ End‑to‑end working AI agent

This project demonstrates **real AI workflow engineering**, not a demo.

---

Happy building 🚀

