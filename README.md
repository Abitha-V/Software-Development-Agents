# 🤖 Software Development Multi-Agent System

<p align="left">
  <img src="https://img.shields.io/badge/Flowise-Agentic%20AI-blue?style=flat-square"/>
  <img src="https://img.shields.io/badge/Gemini%202.5%20Flash-Google%20AI-orange?style=flat-square&logo=google"/>
  <img src="https://img.shields.io/badge/Pattern-Supervisor%20%2B%20Workers-green?style=flat-square"/>
  <img src="https://img.shields.io/badge/Loops-Max%205%20iterations-purple?style=flat-square"/>
  <img src="https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square"/>
</p>

A **multi-agent agentic AI workflow** built with Flowise that simulates a real software development team — a **Supervisor**, a **Software Engineer**, and a **Code Reviewer** — collaborating in an automated loop to deliver production-quality code with review.

---

## 📌 What It Does

Given a user's software development request, the system:

1. Routes the task to a **Supervisor** LLM that decides which specialist to invoke next
2. Dispatches to a **Software Engineer** agent to write or improve the code
3. Dispatches to a **Code Reviewer** agent to audit quality, completeness, and correctness
4. Loops both agents back through the Supervisor until the task is complete
5. Synthesizes a **final answer** with full code, review findings, and improvements

---

## 🏗️ Architecture

### Flowise Flow Graph

![Software Development Multi-Agent Flow](screenshots/flowise-flow.png)

### Flow Breakdown

```
User Input (Chat)
      │
      ▼
┌─────────────┐     Structured JSON output
│  Supervisor │ ──► {next: "SOFTWARE ENGINEER" | "Code Reviewer" | "FINISH"}
│   (Gemini)  │ ──► {instructions: "<specific subtask>"}
└─────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│     Condition Router                │
│  if next == "SOFTWARE ENGINEER" ──► Software Engineer Agent ──► Loop → Supervisor
│  if next == "Code Reviewer"     ──► Code Reviewer Agent     ──► Loop → Supervisor
│  else (FINISH)                  ──► Final Answer Synthesizer
└─────────────────────────────────────┘
      │
      ▼
┌──────────────────────────┐
│  Generate Final Answer   │
│  Full code + review +    │
│  improvements            │
└──────────────────────────┘
```

---

## 🧠 Agent Roles

### Supervisor
- **Model:** Gemini 2.5 Flash
- **Role:** Orchestrator — reads the full conversation history and decides which worker acts next
- **Output:** Structured JSON with `next` (enum: SOFTWARE ENGINEER / Code Reviewer / FINISH) and `instructions` (specific subtask for the next worker)
- **Memory:** All messages — full conversation history retained across the loop

### Software Engineer
- **Model:** Gemini 2.5 Flash
- **Role:** Full-stack developer — writes, implements, or improves code based on supervisor instructions
- **Memory:** All messages — aware of what the reviewer flagged in previous iterations
- **Loop:** Returns output to Supervisor (max 5 iterations)

### Code Reviewer
- **Model:** Gemini 2.5 Flash
- **Role:** Quality gatekeeper — checks code for completeness, missing features, bugs, and standards
- **Memory:** All messages — reviews in context of what the engineer built
- **Loop:** Returns findings to Supervisor (max 5 iterations)

### Final Answer Synthesizer
- **Model:** Gemini 2.5 Flash
- **Role:** Consolidates the entire multi-turn dialogue into a clean, complete final output including full code, all improvements made, and the final review verdict

---

## 🔄 Flow State

The system uses **runtime flow state** to coordinate routing between agents:

| State Key | Type | Purpose |
|---|---|---|
| `next` | string | Name of the next worker to invoke (set by Supervisor, read by Condition node) |
| `instructions` | string | Specific subtask instructions passed from Supervisor to the next worker |

State persists across the loop so the Supervisor's routing decision is available to the Condition node at every iteration.

---

## ⚙️ Technical Details

| Parameter | Value |
|---|---|
| Orchestration | Flowise Agentflow |
| LLM | Google Gemini 2.5 Flash |
| Temperature | 0.9 |
| Memory Type | All Messages (full history) |
| Max Loop Count | 5 iterations per agent |
| Structured Output | JSON (Supervisor only) |
| Input Type | Chat Input |
| State Persistence | Session-scoped flow state |

---

## 📂 Repository Structure

```
software-development-agents/
├── README.md
├── Software_Development_Agents.json   ← Flowise flow (import directly)
├── docs/
│   └── architecture.md                ← Detailed design notes
└── screenshots/
    └── flowise-flow.png               ← Flow graph screenshot (add yours here)
```

---

## 🚀 How to Run

### Prerequisites
- [Flowise](https://github.com/FlowiseAI/Flowise) installed locally or hosted
- Google Gemini API key (Gemini 2.5 Flash)

### Setup

```bash
# Install Flowise
npm install -g flowise

# Start Flowise
npx flowise start
```

1. Open Flowise at `http://localhost:3000`
2. Go to **Agentflows** → **Add New**
3. Click **Load** and import `Software_Development_Agents.json`
4. Add your **Google Gemini API key** in Flowise credentials
5. Click **Save** and then **Deploy**
6. Use the **Chat** interface to submit a software development task

### Example Prompts

```
Build a REST API in Python using FastAPI with CRUD operations for a user management system.
```

```
Write a React component for a data table with sorting, filtering, and pagination.
```

```
Create a SQL query optimiser function in Python that analyses and rewrites slow queries.
```

---

## 💡 Design Decisions

**Why a Supervisor pattern?**
Rather than a fixed pipeline (engineer → reviewer → done), the Supervisor pattern lets the LLM decide dynamically how many iterations are needed and in what order. A simple task might go engineer → FINISH. A complex one might loop engineer → reviewer → engineer → reviewer → FINISH.

**Why structured JSON output from the Supervisor?**
The Condition node needs a deterministic string to route on. Structured output with an enum (`SOFTWARE ENGINEER / Code Reviewer / FINISH`) eliminates ambiguity and makes routing reliable.

**Why all-messages memory?**
Each agent needs full context of what was built and reviewed in previous iterations. Windowed or summarised memory would cause the engineer to repeat work already done or the reviewer to miss previously flagged issues.

**Why max 5 loops?**
Prevents infinite loops on tasks where the Supervisor never emits FINISH. The fallback message prevents a silent failure.

---

## 🗺️ Roadmap

- [ ] Add a **Security Reviewer** agent (OWASP checks, injection vulnerabilities)
- [ ] Add a **Test Writer** agent (generates unit tests for produced code)
- [ ] Connect a **Code Execution** tool so the engineer can run and verify code output
- [ ] Add LangSmith observability for trace logging across agent turns
- [ ] Persist conversation history to Redis for session resume

---

## 👩‍💻 Author

**Abitha V** — MBA (Banking Technology), Pondicherry University

Agentic AI · Data Analytics · BI & Fintech

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin)](https://linkedin.com/in/abithavinayagamurugan)
[![Email](https://img.shields.io/badge/Email-abithabi.v%40gmail.com-red?style=flat-square&logo=gmail)](mailto:abithabi.v@gmail.com)

---

## 📄 License

MIT — free to use, modify, and build on.
