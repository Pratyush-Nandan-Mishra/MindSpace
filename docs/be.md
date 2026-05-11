# MindSpace — Backend Architecture

**Runtime:** Node.js 22 · ES Modules (`"type": "module"`)
**Framework:** Express.js
**Port:** `4433`
**Database:** MongoDB Atlas (Mongoose ODM)

---

## Stack at a Glance

| Layer | Technology |
|---|---|
| Server | Express.js |
| Database | MongoDB Atlas + Mongoose |
| AI Agent | LangGraph (ReAct pattern) |
| LLM | Groq — `llama-3.3-70b-versatile` |
| Memory summarisation | Google Gemini 1.5 Flash |
| Web search | SerpAPI (crisis resource queries only) |
| Auth | Google OAuth 2.0 + Passport.js |
| Sessions | express-session + connect-mongo |
| Tokens | JWT (access + refresh via jsonwebtoken) |
| Rate limiting | Custom per-user DB-backed limiter |

---

## Folder Structure

```
BE/src/
├── agents/         LangGraph ReAct agent (chatAgent.js)
├── config/         DB connection · Passport Google OAuth strategy
├── controllers/    Thin route handlers — delegate to helpers
├── helpers/        Core business logic (chat, crisis, FAQ, JWT, SerpAPI)
├── middleware/     Auth guards · JWT validator · rate limiter · validation
├── models/         Mongoose schemas (8 models)
├── routes/         Express routers mounted under /api
├── scripts/        One-off seeds (FAQ seed from faqs.json)
├── tools/          LangChain Tool classes used by the agent
├── utils/          ApiError · ApiResponse · AsyncHandler
├── validators/     express-validator rule sets
└── wrapper/        Long-term memory read/write (memory.js)
```

---

## Entry Point — server.js

```
startup
  → connect MongoDB (config/database.js)
  → configure express-session (MongoStore — sessions live in DB)
  → initialise Passport + Google OAuth strategy
  → register all routes under /api prefix
  → global error handler (Mongoose ValidationError / CastError → 400)
  → listen on PORT 4433
```

---

## ★ Feature Highlights

### 1 — LangGraph AI Agent (ReAct pattern)
The chat pipeline uses a **LangGraph ReAct agent** (`agents/chatAgent.js`) that decides at runtime whether to:
- Check the **FAQ tool** first (MongoDB full-text search over curated Q&A pairs)
- Fall through to the **LLM tool** (Groq) if no FAQ match is confident enough

This means frequent, predictable questions are answered instantly from cache without spending an LLM call.

### 2 — 3-Tier Crisis Detection (`helpers/crisis.helper.js`)
Every user message is scanned **before** the LLM is called:

| Tier | Triggers | Action |
|---|---|---|
| `crisis` | Suicidal ideation, self-harm intent | Prepend helplines (iCall, Vandrevala, NIMHANS, Crisis Text Line) + empathy lead |
| `high` | Self-harm language, overdose references | Prepend helplines + concern message |
| `moderate` | Panic attacks, "can't cope", hopelessness | Prepend gentle grounding response |
| `none` | Everything else | No prefix — normal LLM response |

Crisis-level messages are **never saved** to long-term memory or FAQ (privacy + trauma protection).

### 3 — Long-Term Memory (`wrapper/memory.js` + Gemini)
After every non-crisis exchange, Gemini 1.5 Flash summarises the conversation turn into 1–2 sentences and appends it to the user's `ChatMemory` document. On the next message, these summaries are injected into the LLM system prompt so MindSpace "remembers" past context across sessions.

### 4 — Ethical Safeguards (enforced in system prompt)
Hardcoded into `llmTool.js` PROMPT constant — the LLM is instructed to:
- **Never diagnose** any mental health condition
- **Never discuss medication**
- Redirect off-topic questions back to emotional wellbeing
- Include a disclaimer naturally when relevant
- Never ask for personally identifiable information

### 5 — Role-Based Access Control
`User.role` field is either `"user"` or `"admin"`. Admin routes (`/api/admin`, `/api/stats`, `/api/faq`) are protected by an additional role-check middleware on top of the standard auth guard.

### 6 — Per-User Rate Limiting
`middleware/ratelimit.middleware.js` tracks message counts in the `MessageRateLimit` MongoDB collection. When a user hits the limit, the API returns a `429` with a custom `botResponse` message that the frontend renders in the chat thread.

### 7 — FAQ System + Seeding
`tools/faqTool.js` does a MongoDB text search on the `FAQ` collection. The collection is pre-seeded via `scripts/seed-faq.js` (reads `scripts/faqs.json`). New Q&A pairs are also auto-saved from non-crisis LLM conversations, so the FAQ grows organically.

---

## AI Chat Pipeline (full trace)

```
POST /api/chat/chat
  └─ chat.controller.js           validates session, extracts userId
       └─ chat.helper.js          loads/creates Conversation document
            └─ chatAgent.js       LangGraph ReAct agent loop
                 ├─ faqTool.js    MongoDB text search → return if match found
                 └─ llmTool.js    main LLM path
                      ├─ crisis.helper.js     detect tier, build prefix
                      ├─ wrapper/memory.js    load long-term summaries → inject into prompt
                      ├─ serpapi.helper.js    optional web search (crisis resource queries)
                      ├─ Groq API call        llama-3.3-70b-versatile
                      ├─ [prefix + response]  crisis helplines prepended if needed
                      └─ (if not crisis)
                           ├─ Gemini summarise → append to ChatMemory
                           └─ saveToFAQ        persist Q&A pair
```

---

## Data Models

| Model | Key fields | Purpose |
|---|---|---|
| `User` | `googleId`, `email`, `role`, `picture` | Auth identity + role |
| `Conversation` | `userId`, `messages[]`, `title` | Per-user chat history |
| `ChatMemory` | `userId`, `summaries[]` | Long-term memory (Gemini summaries) |
| `FAQ` | `question`, `answer` (text-indexed) | Cached knowledge base |
| `MessageRateLimit` | `userId`, `count`, `windowStart` | Rate limit tracking |
| `LLMCall` | `userId`, `model`, `tokens`, `createdAt` | Admin usage analytics |
| `Feedback` | `userId`, `message`, `rating` | User feedback submissions |
| `Billing` | `userId`, `plan` | Future billing / plan tier |

---

## API Routes

| Method | Path | Auth | Description |
|---|---|---|---|
| `GET` | `/api/auth/google` | — | Redirect to Google OAuth |
| `GET` | `/api/auth/google/callback` | — | OAuth callback, sets session |
| `GET` | `/api/auth/me` | User | Returns current user object |
| `GET` | `/api/auth/logout` | User | Destroys session |
| `POST` | `/api/chat/chat` | User | Send a message, get AI reply |
| `GET` | `/api/chat/conversations` | User | List all conversations |
| `GET` | `/api/chat/conversation/:id` | User | Load a single conversation |
| `DELETE` | `/api/chat/conversation/:id` | User | Delete a conversation |
| `DELETE` | `/api/chat/conversations/clear` | User | Delete all conversations |
| `GET` | `/api/stats` | Admin | Usage statistics |
| `GET/POST/DELETE` | `/api/faq` | Admin | Manage FAQ entries |
| `GET` | `/api/admin/users` | Admin | List all users |

---

## Environment Variables

```env
# Server
PORT=4433
FRONTEND_URL=http://localhost:8081

# Database
MONGODB_URI=mongodb+srv://<user>:<pass>@cluster.mongodb.net/<db>

# Google OAuth  (console.cloud.google.com)
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# Secrets  (generate with: node -e "console.log(require('crypto').randomBytes(64).toString('hex'))")
JWT_SECRET=
JWT_REFRESH_SECRET=
SESSION_SECRET=

# AI APIs
GROQ_API_KEY=          # console.groq.com
GEMINI_API_KEY=        # aistudio.google.com
SERP_API_PRIVATE_KEY=  # serpapi.com
```
