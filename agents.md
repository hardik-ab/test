# agents.md — Multi-Agent System

> Defines how Claude Code should split work across agents when building or modifying this project.

---

## 🏗️ Agent Overview

```
User Request
     │
     ▼
ORCHESTRATOR
     │
  ┌──┴───────────┬──────────────┐
  ▼              ▼              ▼
FETCHER       ANALYST        DESIGNER
(server.js)  (prompt.md)    (index.html)
```

---

## 👤 Agent Definitions

---

### 🧠 ORCHESTRATOR
**Role:** Reads the task and CLAUDE.md, assigns work to the right agents, never writes code directly.

**Rules:**
- Always start by reading CLAUDE.md
- For any new feature: FETCHER first, then ANALYST, then DESIGNER
- If only UI changes: DESIGNER only
- If only API/data changes: FETCHER only

---

### 📡 FETCHER
**Role:** Owns `server.js`. Responsible for all YouTube API calls and the Express backend.

**Responsibilities:**
- YouTube Data API v3 calls (channel stats, videos, comments, captions)
- Express routes (`GET /`, `POST /analyze`)
- Passing clean structured data to the ANALYST

**Rules:**
- All API keys come from `.env` only — never hardcoded
- Every API call must have try/catch with a meaningful error message
- Return raw structured data only — no formatting, no opinions

**Output shape for `/analyze`:**
```json
{
  "channel": { "name": "", "subscribers": 0, "totalViews": 0, "videoCount": 0 },
  "videos": [{ "title": "", "views": 0, "likes": 0, "comments": 0 }],
  "topComments": ["comment1", "comment2"],
  "transcripts": ["transcript text from video 1", "..."]
}
```

---

### 🧠 ANALYST
**Role:** Owns `prompt.md`. Writes and refines the Claude prompt that turns raw data into the analysis report.

**Responsibilities:**
- Craft the system prompt that instructs Claude on tone, structure, and output format
- Define exactly what sections the report must contain
- Ensure the prompt produces consistent, structured output every time

**Report sections the prompt must produce:**
```
📊 Channel Overview
🎯 Content Quality Score (X/10 with reasoning)
🔥 What's Working
💀 What's Not Working
🗣️ Audience Sentiment (from comments)
📝 Key Themes & Topics
⚡ Best Performing Format
💡 Recommendations
```

**Rules:**
- Prompt must instruct Claude to be direct and opinionated — not generic
- Tone: smart, slightly witty, like a brutally honest media consultant
- Output must always use the section headers above so the UI can parse them

---

### 🎨 DESIGNER
**Role:** Owns `index.html`. Builds and maintains the entire frontend as a single self-contained file.

**Responsibilities:**
- Input field for Channel ID + Analyze button
- Loading state while analysis runs
- Render the report in a clean, readable layout
- Error states (invalid channel, API failure)

**Rules:**
- Tailwind CSS via CDN only — no local CSS files
- Dark theme (`bg-gray-950` background)
- Report sections should each be a distinct card
- No page reloads — use `fetch()` to call `/analyze`
- Show a pulsing skeleton loader while waiting for the response

---

## 🔄 Task Flow Examples

**"Add a new data point to the analysis"**
```
ORCHESTRATOR → FETCHER (add API call) → ANALYST (update prompt) → DESIGNER (update UI if needed)
```

**"Fix the UI layout"**
```
ORCHESTRATOR → DESIGNER only
```

**"The transcript fetch is breaking"**
```
ORCHESTRATOR → FETCHER only
```

---

## 🚨 Escalation Rules

Stop and ask the user when:
- YouTube API quota is exceeded
- A channel has no captions/transcripts available
- The Claude API returns an unexpected format
