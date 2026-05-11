# MindSpace — My Contribution: AI Engine & Intelligence Layer
### Backend Developer 1

---

## What I worked on

I built the **brain of MindSpace** — everything that makes the AI smart, safe, and context-aware. This includes the AI agent pipeline, the crisis detection system, the long-term memory system, the FAQ knowledge base, and the integration with three different AI APIs.

The backend team split the work into two parts: my teammate handled the authentication, database models, and API routes. I focused on making the AI actually work well.

---

## Technologies I used

| Technology | Why I used it |
|---|---|
| LangGraph | Framework to build the AI agent with a decision loop (ReAct pattern) |
| LangChain | Tool abstraction layer that plugs into LangGraph |
| Groq API | Ultra-fast LLM inference — runs `llama-3.3-70b-versatile` |
| Google Gemini 1.5 Flash | Lightweight, fast model used specifically for memory summarization |
| SerpAPI | Live web search — used to fetch real crisis helpline info |
| MongoDB (text search) | Powers the FAQ lookup tool |
| Node.js ES Modules | All agent and tool files use `import/export` syntax |

---

## Feature 1: The LangGraph ReAct AI Agent

This is the core of the whole product. Instead of just sending the user's message directly to an LLM and returning whatever it says, I built a proper **AI agent** using LangGraph.

**What is a ReAct agent?**
ReAct stands for "Reason and Act." The agent doesn't just answer — it first *reasons* about which tool to use, then *acts* by calling that tool, then decides if the answer is good enough or if it needs to try something else.

**How the agent works:**

```
User sends a message
       ↓
LangGraph agent decides: "Do I have a quick answer in FAQ?"
       ↓
  [FAQ Tool] → MongoDB text search over stored Q&A pairs
       ↓
  Match found? → Return FAQ answer immediately (no LLM call, saves cost & time)
  No match?   → Fall through to LLM Tool
       ↓
  [LLM Tool] → Full pipeline (see below)
```

**Why this matters:** Common mental health questions (What is anxiety? How do I calm down?) are answered instantly from the FAQ cache without spending an API call. Only unique or complex questions go to the LLM.

**Key file:** `agents/chatAgent.js`

---

## Feature 2: 3-Tier Crisis Detection

Every single user message is scanned for crisis signals **before** the LLM ever sees it. I built a 4-level classification system in `helpers/crisis.helper.js`.

| Tier | What triggers it | What happens |
|---|---|---|
| `crisis` | Suicidal thoughts, self-harm intent, phrases like "I want to end it" | Helplines (iCall, Vandrevala Foundation, NIMHANS, Crisis Text Line) are prepended to the response. The reply starts with empathy, not advice. |
| `high` | Self-harm language, overdose mentions | Same helplines prepended, with a concern message |
| `moderate` | Panic attack language, "I can't cope", hopelessness | A gentle grounding response is added |
| `none` | Everything else | Normal LLM flow, no modification |

**Important safety rule I implemented:** Crisis-level messages are **never saved** to long-term memory or the FAQ database. This protects user privacy and ensures traumatic content is not repurposed as training data.

**Why this is better than just prompting the LLM:** The LLM might fail. A pattern-matching classifier runs deterministically and never misses a keyword. Safety-critical behavior should not depend on probabilistic model outputs.

---

## Feature 3: Long-Term Memory with Gemini

Standard chatbots forget everything between sessions. MindSpace remembers.

**How I built this:**

After every non-crisis conversation turn, I send the exchange to **Google Gemini 1.5 Flash** with this instruction: "Summarize this conversation turn in 1–2 sentences, focusing on what the user shared emotionally." Gemini returns a compact summary.

That summary is appended to the user's `ChatMemory` document in MongoDB.

On the user's next visit (even days later), when they send a new message, the system:
1. Reads all their past summaries from `ChatMemory`
2. Injects them into the Groq LLM's system prompt as context
3. Now the LLM "knows" what the user has shared before

**Example:**
- Session 1: User talked about work stress and difficulty sleeping
- Session 2: User says "I'm feeling better today"
- MindSpace can respond: "That's great to hear — last time you mentioned trouble sleeping from work stress. Did something change?"

**Key files:** `wrapper/memory.js`, `helpers/chat.helper.js`

---

## Feature 4: FAQ System — Growing Knowledge Base

I built a dual-mode FAQ system:

**Pre-seeded knowledge:** I created a JSON file (`scripts/faqs.json`) with common mental health Q&A pairs. A seed script (`scripts/seed-faq.js`) loads these into MongoDB at startup with a text index on the `question` and `answer` fields.

**Auto-growing:** Every non-crisis LLM response is automatically saved back to the FAQ collection. So when the LLM gives a good answer to something new, that answer becomes instantly available as a cached response for the next user who asks something similar. The FAQ grows without anyone manually adding entries.

**FAQ Tool:** The LangChain `faqTool.js` does a MongoDB `$text` search. If the relevance score is high enough, it returns the cached answer. If not, it passes control to the LLM.

---

## Feature 5: Ethical Safeguards in the System Prompt

I hardcoded the ethical behavior directly into the LLM's system prompt in `tools/llmTool.js`. No matter what the user asks, the LLM is permanently instructed to:

- **Never diagnose** any mental health condition (only a licensed professional can do that)
- **Never recommend or discuss medication** of any kind
- **Redirect off-topic requests** back to emotional wellbeing topics
- **Include a disclaimer naturally** when relevant ("I'm an AI and not a replacement for professional help")
- **Never ask for personally identifiable information** — no names, locations, phone numbers

These rules are part of the prompt constant, not configurable at runtime, so they cannot be bypassed.

---

## Feature 6: SerpAPI — Live Crisis Resource Search

When the crisis detection flags a message as `crisis` tier, I added an optional step: use **SerpAPI** to do a live web search for up-to-date crisis resources in the user's region. This ensures the helpline numbers shown are current and relevant.

**Key file:** `helpers/serpapi.helper.js`

---

## The Full AI Pipeline (what happens in every chat message)

```
POST /api/chat/chat
   ↓
chat.helper.js — loads or creates the Conversation document
   ↓
chatAgent.js — LangGraph ReAct agent starts
   ↓
  [FAQ Tool]  → MongoDB text search
    └─ match found → return immediately
    └─ no match →
  [LLM Tool]
    ├─ crisis.helper.js     → classify message (crisis / high / moderate / none)
    ├─ wrapper/memory.js    → load past summaries → inject into system prompt
    ├─ serpapi.helper.js    → (only for crisis tier) fetch live resources
    ├─ Groq API call        → llama-3.3-70b-versatile generates response
    ├─ prepend crisis prefix if needed
    └─ if NOT crisis:
         ├─ Gemini → summarize turn → save to ChatMemory
         └─ save Q&A pair to FAQ collection
```

---

## Key Files I Owned

```
BE/src/
├── agents/chatAgent.js             LangGraph ReAct agent
├── tools/
│   ├── faqTool.js                  LangChain tool — MongoDB FAQ search
│   └── llmTool.js                  LangChain tool — Groq LLM + safeguards
├── helpers/
│   ├── crisis.helper.js            3-tier crisis classification
│   ├── chat.helper.js              Conversation load/create + pipeline entry
│   └── serpapi.helper.js           Live web search for crisis resources
└── wrapper/memory.js               Gemini summarization + memory inject
```

---

## Summary

I built the intelligence layer that makes MindSpace more than a chatbot wrapper. The ReAct agent, the crisis safety system, and the memory that persists across sessions are the features that make this project genuinely interesting from a technical standpoint. Each one required integrating a different AI API (Groq, Gemini, SerpAPI) and making sure they work together reliably without any single point of failure.
