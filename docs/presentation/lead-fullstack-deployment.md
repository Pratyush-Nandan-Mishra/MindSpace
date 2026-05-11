# MindSpace — My Contribution: System Architecture, Key Features & Deployment
### Full Stack Lead / Project Lead

---

## What I worked on

As the team lead, I had two responsibilities. First, I helped architect the overall system and made sure all the pieces — frontend, backend, database, and AI — fit together correctly. Second, I handled everything related to **deployment** — getting the project off localhost and running live on the internet.

I also helped both the frontend and backend teams when they got stuck, reviewed the integration points, and made sure the final product was demo-ready.

---

## Technologies I oversaw and used

| Area | Technology |
|---|---|
| Backend runtime | Node.js 22 + Express.js |
| Frontend | React 19 + Vite 7 |
| Database (cloud) | MongoDB Atlas |
| AI (primary LLM) | Groq — llama-3.3-70b-versatile |
| AI (memory) | Google Gemini 1.5 Flash |
| Auth provider | Google OAuth 2.0 (Google Cloud Console) |
| Deployment — Backend | Render (Node.js web service) |
| Deployment — Frontend | Vercel (static Vite build) |
| Environment management | `.env` files + cloud environment variable panels |

---

## The Full System Architecture

```
┌─────────────────────────────────┐
│          USER'S BROWSER         │
│   Vercel CDN → React 19 + Vite  │
└────────────────┬────────────────┘
                 │ HTTPS
                 ▼
┌─────────────────────────────────┐
│        RENDER (Backend)         │
│     Node.js 22 + Express        │
│     Port 4433 → /api/*          │
│                                 │
│  ┌──────────┐  ┌─────────────┐  │
│  │ Passport │  │ express-    │  │
│  │ OAuth    │  │ session     │  │
│  └──────────┘  └─────────────┘  │
│                                 │
│  ┌──────────────────────────┐   │
│  │   LangGraph ReAct Agent  │   │
│  │  FAQ Tool → LLM Tool     │   │
│  └──────────────────────────┘   │
└────┬──────────┬──────────┬──────┘
     │          │          │
     ▼          ▼          ▼
MongoDB      Groq API   Gemini API
 Atlas       (LLM)      (Memory)
```

---

## Major Features of MindSpace (Full Overview)

### 1 — Google OAuth Sign-In
Users sign in with their Google account — no passwords needed. The backend handles the OAuth handshake with Google, creates a session, and issues JWT tokens. Both the session cookie and JWT are used together for secure, persistent login.

### 2 — LangGraph ReAct AI Agent
The heart of the product. Instead of just forwarding messages to an LLM, we built a proper AI agent using LangGraph. The agent first checks a local FAQ database, and only calls the external Groq LLM if no good match is found. This saves API cost and makes common questions answer instantly.

### 3 — 3-Tier Crisis Detection
Every user message is scanned before the LLM sees it. If suicidal ideation or self-harm language is detected, real helpline numbers (iCall, Vandrevala Foundation, NIMHANS) are prepended to the response automatically. No LLM decision-making is involved in this safety check — it's a deterministic classifier.

### 4 — Long-Term Memory with Gemini
After each conversation turn, Google Gemini 1.5 Flash summarizes the exchange into 1–2 sentences. Those summaries are stored per user and injected into the AI's system prompt on every future visit — so MindSpace genuinely remembers what users have shared across sessions.

### 5 — Self-Growing FAQ System
Every non-crisis LLM response is saved to the FAQ collection. Over time, the FAQ grows automatically and more questions get answered from cache rather than from the LLM.

### 6 — Animated Landing Page
A full 8-section public marketing page with scroll-triggered animations, a live demo chat in the hero section, an infinite testimonial marquee, and a complete ethical safeguards section.

### 7 — Full Chat UI with Typewriter Effect
Real-time chat interface with character-by-character typewriter animation, stop-generation button, quick-reply chips, copy and text-to-speech for every AI message.

### 8 — Find a Professional + Mock Call UI
A button in the chat UI opens a directory of 6 mental health professionals. Clicking Contact opens a full-screen simulated voice call UI with pulsing rings, a live call timer, mute/speaker/end-call controls.

### 9 — Admin Dashboard + Analytics
Admin-only pages showing user lists, conversation counts, LLM call logs, token usage, and rate limit hit counts.

### 10 — Per-User Rate Limiting
MongoDB-backed rate limiter that counts messages per user per time window. When a user hits the limit, they get a friendly message in the chat UI instead of a raw error.

---

## Deployment — What I Did

### Step 1: MongoDB Atlas Setup
- Created a free-tier Atlas cluster
- Created a database user with a strong password
- Whitelisted `0.0.0.0/0` so both local dev and production servers can connect
- Copied the connection string (`MONGODB_URI`) for use in env variables

### Step 2: Google Cloud Console Setup
- Created a new project in Google Cloud Console
- Enabled the **Google+ API** and **OAuth 2.0**
- Created OAuth credentials (Client ID + Client Secret)
- Added authorized redirect URIs for both local dev and the production backend URL:
  ```
  http://localhost:4433/api/auth/google/callback   ← local dev
  https://mindspace-api.onrender.com/api/auth/google/callback  ← production
  ```

### Step 3: AI API Keys
- **Groq API key** — created at console.groq.com, free tier, gives access to llama-3.3-70b-versatile
- **Gemini API key** — created at aistudio.google.com, free tier, used for memory summarization
- **SerpAPI key** — created at serpapi.com, used for live crisis resource web search

### Step 4: Backend Deployment on Render
- Connected the GitHub repository to Render
- Set the root directory to `BE/`
- Set the build command: `npm install`
- Set the start command: `node src/server.js`
- Added all environment variables in the Render dashboard:
  ```
  PORT=4433
  MONGODB_URI=mongodb+srv://...
  GOOGLE_CLIENT_ID=...
  GOOGLE_CLIENT_SECRET=...
  JWT_SECRET=...
  JWT_REFRESH_SECRET=...
  SESSION_SECRET=...
  GROQ_API_KEY=...
  GEMINI_API_KEY=...
  SERP_API_PRIVATE_KEY=...
  FRONTEND_URL=https://mindspace.vercel.app
  ```
- Deployed and verified the backend API was responding at the Render URL

### Step 5: Frontend Deployment on Vercel
- Connected the GitHub repository to Vercel
- Set the root directory to `FE/`
- Set the build command: `npm run build` (Vite builds to `dist/`)
- Set the output directory: `dist`
- Added one environment variable:
  ```
  VITE_API_URL=https://mindspace-api.onrender.com/api
  ```
- Updated `ChatMessage.jsx` to use `import.meta.env.VITE_API_URL` instead of the hardcoded localhost URL
- Deployed and verified the frontend was loading and connecting to the backend

### Step 6: CORS Configuration
Updated the backend `server.js` CORS settings to allow requests from the production Vercel domain:
```js
cors({
  origin: process.env.FRONTEND_URL,
  credentials: true   // required for session cookies to work cross-origin
})
```

### Step 7: Final Testing
- Tested the full Google OAuth flow on production (localhost redirect URIs don't work on prod)
- Tested the chat feature with a real message — confirmed AI agent, FAQ, and memory all working
- Tested the crisis detection by sending a test message
- Verified admin routes are accessible only with an admin account
- Confirmed rate limiting works after the message threshold is hit

---

## Key Decisions I Made as Lead

**Why MongoDB Atlas over a local database?**
Atlas is free-tier, always available, and doesn't go down when the dev's laptop sleeps. For a cloud deployment, a cloud database is the right choice.

**Why Groq for the LLM?**
Groq runs llama-3.3-70b-versatile with extremely low latency — responses come back in 1–2 seconds instead of 5–10 seconds with other providers. For a chat app, response speed directly affects user experience.

**Why Gemini for memory summarization?**
Gemini 1.5 Flash is fast and cheap for a simple summarization task. We did not need a powerful model for this — just something that can compress a conversation into 2 sentences reliably.

**Why separate frontend and backend deployments?**
This is the industry standard. It allows independent scaling — if the backend gets more traffic, we scale only the backend. It also means frontend updates don't require backend restarts.

---

## Summary

My role was to make sure the whole project actually works — not just on localhost, but in the real world. Setting up three AI API accounts, configuring Google OAuth for production, managing environment variables securely, and coordinating the final integration between the frontend and backend were all on me. The product is live because of the deployment work, and the overall quality of MindSpace reflects the architectural decisions made at the start.
