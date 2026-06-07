# 🤖 N8N AI Agent Workshop — Task Manager + Exam Paper Generator

A hands-on workshop project demonstrating how to **deploy a real web server**, expose it as a **REST API**, and connect it to **N8N AI Agents** as tools.

Two complete workflows are included:

| Workflow | Trigger | What it does |
|---|---|---|
| **Task Manager Agent** | Chat / Telegram | Add, list, complete, and delete tasks via natural language |
| **Exam Question Paper Bot** | Telegram | Send a topic → AI generates questions → returns a PDF |

> 🌐 **Live Server (Reference):** [`https://task-management-898555939324.europe-west1.run.app/docs`](https://task-management-898555939324.europe-west1.run.app/docs)

---

## 📐 Architecture Overview

### Workflow 1 — Task Manager AI Agent

```
User (Chat / Telegram)
        │
        ▼
   N8N AI Agent  ◄──── Google Gemini 1.5 Flash
        │         ◄──── Window Buffer Memory
        │
   ┌────┼──────────────────────┬──────────────┐
   ▼    ▼         ▼            ▼              ▼
 POST  GET      PATCH        DELETE      Google Calendar
/tasks /tasks /tasks/{id}  /tasks/{id}    create: event
        │
        ▼
  FastAPI Server
  (Google Cloud Run)
  SQLite Database
```

### Workflow 2 — Telegram Exam Question Paper Bot

```
Telegram Message
  (topic / subject)
        │
        ▼
  N8N AI Chain
  (Gemini Flash)
        │
        ▼
  Generate Questions
  + Answers (Markdown)
        │
        ▼
  Convert to PDF
        │
        ▼
  Send PDF via Telegram
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Web Server | Python FastAPI + SQLite |
| Deployment | Google Cloud Run |
| Workflow Engine | N8N (self-hosted or cloud) |
| AI Model | Google Gemini 1.5 Flash |
| Interfaces | N8N Chat UI + Telegram Bot |
| PDF Generation | N8N + Markdown-to-PDF conversion |

---

## 📁 Repository Structure

```
├── main.py                        # FastAPI server — 4 task endpoints
├── task-manager-agent.json        # N8N Workflow 1 — Task Manager AI Agent
├── exam-question-bot.json         # N8N Workflow 2 — Telegram Exam Paper Bot
└── README.md
```

---

## 🚀 Part 1 — Task Manager AI Agent

### Step 1: Deploy the FastAPI Server

```bash
# Install dependencies
pip install fastapi uvicorn

# Run locally to test
uvicorn main:app --reload --port 8000

# Open interactive API docs
open http://localhost:8000/docs
```

Deploy to **Google Cloud Run** or **Railway** to get a public HTTPS URL.

**Live reference deployment:**
```
https://task-management-898555939324.europe-west1.run.app/docs
```

### Step 2: API Endpoints

| Method | Endpoint | Agent uses it when… |
|---|---|---|
| `POST` | `/tasks` | User says "add" / "create" / "save" |
| `GET` | `/tasks` | User asks "what tasks do I have?" |
| `PATCH` | `/tasks/{id}` | User says "mark as done" / "completed" |
| `DELETE` | `/tasks/{id}` | User says "delete" / "remove" |

### Step 3: Import the N8N Workflow

1. Open N8N → **Workflows → Import from File**
2. Select `task-manager-agent.json`
3. Replace these two credential placeholders:

| Placeholder | Replace with |
|---|---|
| `REPLACE_WITH_GEMINI_CREDENTIAL_ID` | Your Google Gemini API credential |
| `REPLACE_WITH_TELEGRAM_CREDENTIAL_ID` | Your Telegram Bot Token *(optional)* |

### Step 4: Test with Chat UI

The **Chat Trigger** is enabled by default. Click **"Chat"** in the N8N toolbar to test immediately without any Telegram setup.

**Try these prompts:**
```
"Add submit assignment to my list"
"What tasks do I have?"
"Mark submit assignment as done"
"Delete the groceries task"
"What's still pending?"
```

### Step 5: Enable Telegram *(optional)*

1. Create a bot via [@BotFather](https://t.me/BotFather) → copy the token
2. In N8N, add a **Telegram API** credential
3. Enable the **Telegram Trigger** node (disable state → active)
4. Enable the **Send Telegram Reply** node
5. Disable the **Chat Trigger** node
6. Click **Activate** on the workflow

### How the AI Agent Picks Tools

The AI Agent reads your message and decides which HTTP tool to call — no if/else logic needed:

```
"Add buy milk"          →  create_task (POST /tasks)
"What do I need to do?" →  list_tasks  (GET /tasks)
"I finished the report" →  list_tasks → find ID → complete_task (PATCH)
"Remove the gym task"   →  list_tasks → find ID → delete_task (DELETE)
```

**Window Buffer Memory** keeps the last 12 messages in context so the agent remembers task IDs across conversation turns.

---

## 📝 Part 2 — Telegram Exam Question Paper Bot

This is a separate N8N workflow that turns any topic into a formatted exam paper PDF — delivered directly in Telegram.

### How It Works

1. **Student sends a message** to the Telegram bot with a topic or subject
   ```
   Example: "Generate 5 questions on Python functions"
   Example: "Create an exam paper on photosynthesis"
   ```

2. **N8N receives the message** via Telegram Trigger

3. **Gemini AI generates** a structured exam paper with:
   - Questions (MCQ / short answer / descriptive)
   - Model answers for each question

4. **The output is converted to PDF** inside the workflow

5. **Telegram bot sends the PDF** back to the student

### Import and Setup

1. Import `exam-question-bot.json` into N8N
2. Add your **Telegram Bot Token** credential
3. Add your **Google Gemini API** credential
4. Activate the workflow

### Example Telegram Interaction

```
You:  "Generate 5 short answer questions on Machine Learning basics"

Bot:  📄 Here is your exam paper!
      [Sends: ML_Basics_ExamPaper.pdf]
```

The PDF includes questions on the first half and model answers on the second half — ready to print or share.

---

## 🧠 Key Learning Points for Students

| Concept | Where it's shown |
|---|---|
| REST API design | `main.py` — FastAPI endpoints |
| Cloud deployment | Google Cloud Run → public HTTPS URL |
| N8N tool calling | AI Agent using HTTP Request tools |
| LLM reasoning | Agent picks the right tool from natural language |
| Multi-step tool calls | list_tasks → find ID → complete_task |
| Telegram bot integration | Both workflows use Telegram |
| PDF generation from AI | Exam paper bot workflow |

---

## 📚 Workshop Resources

Built for a hands-on **Google Developer Community** workshop session on deploying AI agents with N8N.

- N8N documentation: [docs.n8n.io](https://docs.n8n.io)
- FastAPI documentation: [fastapi.tiangolo.com](https://fastapi.tiangolo.com)
- Google Gemini API: [ai.google.dev](https://ai.google.dev)
- Telegram Bot API: [core.telegram.org/bots](https://core.telegram.org/bots)

---

## 📝 License

MIT — free to use and modify for learning and teaching purposes.
