
<div align="center">

# 🧠 MindSpace
### *Your AI Mental Health Companion*

**Non-clinical emotional support, available 24/7.**  
Powered by LangGraph · Groq · Gemini · Google OAuth

---

![Node](https://img.shields.io/badge/Node.js-22-339933?style=flat-square&logo=node.js&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?style=flat-square&logo=react&logoColor=black)
![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248?style=flat-square&logo=mongodb&logoColor=white)
![Express](https://img.shields.io/badge/Express.js-4-000000?style=flat-square&logo=express&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-7-646CFF?style=flat-square&logo=vite&logoColor=white)
![Tailwind](https://img.shields.io/badge/Tailwind-v4-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-violet?style=flat-square)

</div>

---

## ✨ What is MindSpace?

MindSpace is a full-stack AI-powered mental health support application that provides **safe, non-clinical emotional support** through a conversational interface.

It is built with strict ethical guardrails — it never diagnoses, never prescribes, and automatically detects crisis signals to surface real helplines immediately.

> **This is a project submission for Section 4.1 — AI Mental Health Support Assistant.**

---

## 🌟 Key Features

| Feature | Description |
|---|---|
| 🤖 **AI Companion** | Powered by `llama-3.3-70b-versatile` via Groq — warm, empathetic, non-clinical responses |
| 🚨 **Crisis Detection** | 3-tier regex engine detects suicidal ideation / self-harm / distress before the LLM responds |
| 📞 **Real Helplines** | iCall, Vandrevala Foundation, NIMHANS, Crisis Text Line auto-surfaced in crisis moments |
| 🧠 **Long-Term Memory** | Gemini 1.5 Flash summarises each exchange and injects past context into future sessions |
| 📚 **FAQ Cache** | LangGraph agent checks MongoDB FAQ before hitting the LLM — faster + cost-efficient |
| 🔐 **Google OAuth** | One-click sign-in, sessions stored in MongoDB |
| 🩺 **Find a Professional** | In-chat directory of mental health professionals + mock web-call UI |
| 🎨 **Animated Landing Page** | Glassmorphism dark UI with scroll-triggered animations, infinite testimonial marquee |
| 🛡️ **Ethical by Design** | Never diagnoses · Never prescribes · No PII · Crisis messages excluded from memory |

---

## 🏗️ Project Structure

```
work-folder/
├── BE/                  Express.js backend (port 4433)
│   ├── src/
│   │   ├── agents/      LangGraph ReAct agent
│   │   ├── helpers/     Business logic (chat, crisis, FAQ, JWT)
│   │   ├── models/      Mongoose schemas (8 models)
│   │   ├── routes/      REST API routes
│   │   ├── tools/       LangChain tools (FAQ + LLM)
│   │   └── server.js    Entry point
│   ├── .env             ← you create this (see below)
│   └── package.json
│
├── FE/                  React 19 + Vite frontend (port 8081)
│   ├── src/
│   │   ├── landing/     Public marketing page
│   │   ├── pages/       App pages (auth, dashboard, chat)
│   │   ├── components/  Sidebar, NavBar, ProtectedRoute
│   │   └── App.jsx      Router + auth wiring
│   └── package.json
│
├── docs/
│   ├── be.md            Backend architecture deep-dive
│   └── fe.md            Frontend architecture deep-dive
│
├── .gitignore
└── README.md            ← you are here
```

---

## ⚙️ Prerequisites

Make sure you have the following **before starting**:

- **Node.js 18+** — [nodejs.org](https://nodejs.org)
- **npm 9+** — bundled with Node
- **MongoDB Atlas account** — [mongodb.com/atlas](https://www.mongodb.com/atlas) (free tier works)
- **Google Cloud Console project** — for OAuth credentials
- **Groq API key** — [console.groq.com](https://console.groq.com) (free)
- **Gemini API key** — [aistudio.google.com](https://aistudio.google.com) (free)
- **SerpAPI key** — [serpapi.com](https://serpapi.com) (free tier)

---

## 🚀 Getting Started

### Step 1 — Clone the repository

```bash
git clone <your-repo-url>
cd MindSpace
```

---

### Step 2 — Set up the Backend

#### 2a. Install dependencies

```bash
cd BE
npm install
```

#### 2b. Create the environment file

Create a file called `.env` inside the `BE/` folder:

```bash
# BE/.env

# ── Server ────────────────────────────────────────────
PORT=4433
FRONTEND_URL=http://localhost:8081

# ── Database ──────────────────────────────────────────
# Get this from MongoDB Atlas → your cluster → Connect → Drivers
MONGODB_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/<dbname>?retryWrites=true&w=majority

# ── Google OAuth ──────────────────────────────────────
# console.cloud.google.com → APIs & Services → Credentials → OAuth 2.0 Client ID
# Authorised redirect URI: http://localhost:4433/api/auth/google/callback
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

# ── Security secrets ──────────────────────────────────
# Generate with: node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
JWT_SECRET=
JWT_REFRESH_SECRET=
SESSION_SECRET=

# ── AI APIs ───────────────────────────────────────────
# Groq: console.groq.com/keys
GROQ_API_KEY=

# Gemini: aistudio.google.com/apikey
GEMINI_API_KEY=

# SerpAPI: serpapi.com/manage-api-key
SERP_API_PRIVATE_KEY=
```

#### 2c. (Optional) Seed the FAQ database

```bash
npm run seed
```

#### 2d. Start the backend

```bash
npm start
```

> Backend is now running at **http://localhost:4433**
> Health check: **http://localhost:4433/api/health**

---

### Step 3 — Set up the Frontend

Open a **new terminal** in the project root.

#### 3a. Install dependencies

```bash
cd FE
npm install
```

#### 3b. Start the dev server

```bash
npm run dev
```

> Frontend is now running at **http://localhost:8081**

---

### Step 4 — Open the app

| URL | What you see |
|---|---|
| `http://localhost:8081` | Landing page |
| `http://localhost:8081/auth` | Login with Google |
| `http://localhost:8081/dashboard` | User chat dashboard |
| `http://localhost:4433/api/health` | Backend health check |

---

## 🔑 Getting Your API Keys

<details>
<summary><strong>MongoDB Atlas (MONGODB_URI)</strong></summary>

1. Go to [mongodb.com/atlas](https://www.mongodb.com/atlas) and create a free account
2. Create a free M0 cluster
3. Click **Connect** → **Drivers** → copy the connection string
4. Replace `<password>` with your database user password
5. Replace `<dbname>` with any name (e.g. `mindspace`)

</details>

<details>
<summary><strong>Google OAuth (GOOGLE_CLIENT_ID + GOOGLE_CLIENT_SECRET)</strong></summary>

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project (or select existing)
3. Enable the **Google+ API** and **Google OAuth2 API**
4. Go to **APIs & Services → Credentials → Create Credentials → OAuth 2.0 Client ID**
5. Application type: **Web application**
6. Authorised redirect URI: `http://localhost:4433/api/auth/google/callback`
7. Copy the **Client ID** and **Client Secret**

</details>

<details>
<summary><strong>Groq API Key (GROQ_API_KEY)</strong></summary>

1. Go to [console.groq.com](https://console.groq.com)
2. Sign up for a free account
3. Navigate to **API Keys → Create API Key**
4. Copy the key — it won't be shown again

</details>

<details>
<summary><strong>Gemini API Key (GEMINI_API_KEY)</strong></summary>

1. Go to [aistudio.google.com](https://aistudio.google.com)
2. Click **Get API key**
3. Create a new API key in a Google Cloud project

</details>

<details>
<summary><strong>SerpAPI Key (SERP_API_PRIVATE_KEY)</strong></summary>

1. Go to [serpapi.com](https://serpapi.com)
2. Sign up for a free account (100 searches/month free)
3. Copy your private API key from the dashboard

</details>

<details>
<summary><strong>JWT + Session Secrets</strong></summary>

Run this in your terminal to generate three secure secrets:

```bash
node -e "
  const crypto = require('crypto');
  console.log('JWT_SECRET=' + crypto.randomBytes(64).toString('hex'));
  console.log('JWT_REFRESH_SECRET=' + crypto.randomBytes(64).toString('hex'));
  console.log('SESSION_SECRET=' + crypto.randomBytes(64).toString('hex'));
"
```

Paste the output values directly into your `.env`.

</details>

---

## 🧰 Development Scripts

### Backend (`BE/`)

```bash
npm start        # Start with node (production-like)
npm run dev      # Start with nodemon (auto-restarts on file change)
npm run seed     # Seed the FAQ database from scripts/faqs.json
```

### Frontend (`FE/`)

```bash
npm run dev      # Start Vite dev server with HMR
npm run build    # Production build → FE/dist/
npm run preview  # Preview the production build locally
npm run lint     # Run ESLint
```

---

## 🛡️ Ethical Safeguards

MindSpace is built with the following mandatory safeguards:

- **Never diagnoses** — enforced in the LLM system prompt
- **Never recommends medication** — enforced in the LLM system prompt
- **Crisis detection** — 3-tier regex runs before every LLM call
- **Helplines auto-surfaced** — iCall · Vandrevala · NIMHANS · Crisis Text Line
- **Crisis messages not saved** — excluded from long-term memory and FAQ cache
- **No PII collection** — prompt explicitly instructs the AI never to ask for it
- **Disclaimer** — shown in the input bar, Trust section, and naturally in conversation

---

## 🗂️ Documentation

Detailed architecture docs live in the `docs/` folder:

| File | Contents |
|---|---|
| [`docs/be.md`](docs/be.md) | Backend architecture — agent pipeline, models, routes, env vars |
| [`docs/fe.md`](docs/fe.md) | Frontend architecture — routing, features, component map, animations |

---

## 🤝 Tech Stack Summary

```
Frontend                          Backend
─────────────────────             ─────────────────────────────────
React 19 + Vite 7                 Node.js 22 (ES Modules)
Tailwind CSS v4                   Express.js
Motion v12 (animations)           MongoDB Atlas + Mongoose
React Router DOM v7               LangGraph (ReAct agent)
Ant Design (dark theme)           Groq — llama-3.3-70b-versatile
react-markdown                    Google Gemini 1.5 Flash
Ant Design Icons                  Google OAuth 2.0 + Passport.js
                                  JWT + express-session
                                  SerpAPI
```

---

<div align="center">

**MindSpace — because you deserve to be heard.**

*Built with care · Not a replacement for professional mental health care*

</div>
