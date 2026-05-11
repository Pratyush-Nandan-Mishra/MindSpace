# MindSpace — Frontend Architecture

**Framework:** React 19 + Vite 7
**Language:** JSX (app pages) + TypeScript (landing page + shared components)
**Styling:** Tailwind CSS v4
**Animations:** Motion v12 — `import { motion } from "motion/react"`
**Icons:** Ant Design Icons
**Port:** `8081` (Vite dev server)

---

## Stack at a Glance

| Layer | Technology |
|---|---|
| UI framework | React 19 |
| Bundler | Vite 7 |
| Styling | Tailwind CSS v4 |
| Animations | Motion (Framer) v12 |
| Routing | React Router DOM v7 |
| Icons | Ant Design Icons |
| Markdown rendering | react-markdown |
| HTTP | native `fetch` (credentials: include) |
| Component library | Ant Design (dark theme) |

---

## Folder Structure

```
FE/src/
├── assets/            Static images
├── components/        Shared components
│   ├── Sidebar.tsx        Chat sidebar (conversations list)
│   ├── NavBar.jsx         Top navigation bar
│   ├── ProtectedRoute.jsx Role-based route guard
│   └── Footer.jsx         Shared footer
├── context/
│   └── AuthContext.jsx    Global auth state (user, loading, login/logout)
├── hooks/
│   └── use-mobile.tsx     Responsive breakpoint hook
├── infra/
│   └── DarkThemeProvider  Ant Design dark theme wrapper
├── landing/               Public marketing page
│   ├── landing.tsx        Page shell (assembles all sections)
│   └── components/
│       ├── Header.tsx
│       ├── HeroSection.tsx
│       ├── FeatureSection.tsx
│       ├── HowItWorks.tsx
│       ├── CompanionSection.tsx
│       ├── TrustSection.tsx
│       ├── TestimonialSection.tsx
│       └── Footer.tsx
├── lib/
│   └── utils.ts           cn() tailwind class merge helper
└── pages/
    ├── Auth.jsx            Login page → Google OAuth redirect
    ├── Home.jsx            Redirect placeholder
    ├── dashboard/          User + admin dashboards
    │   ├── Dashboard.jsx
    │   ├── AdminDashboard.jsx
    │   ├── AdminStats.jsx
    │   ├── Header.jsx
    │   └── ProfileDrawer.jsx
    └── features/
        └── ChatMessage.jsx   ★ Main chat UI
```

---

## Routing

| Path | Component | Guard |
|---|---|---|
| `/` | `LandingPage` | Public |
| `/auth` | `Auth` | Public — redirects to `/dashboard` if already logged in |
| `/dashboard` | `Dashboard` | `ProtectedRoute` (any user) |
| `/admin/dashboard` | `AdminDashboard` | `ProtectedRoute` (admin role) |
| `/admin/stats` | `AdminStats` | `ProtectedRoute` (admin role) |
| `*` | `LandingPage` | Fallback |

`ProtectedRoute` reads from `AuthContext` — redirects to `/auth` if unauthenticated, to `/` if role mismatch.

---

## Auth Flow

```
1. User clicks "Get Started Free"
   → GET http://localhost:4433/api/auth/google
   → Google OAuth consent screen
   → Backend sets session cookie + issues JWT

2. App mounts → AuthContext calls GET /api/auth/me
   → Hydrates { user, role, picture } into React state

3. All fetch() calls include credentials: "include"
   → Session cookie sent automatically with every request
```

---

## ★ Feature Highlights

### 1 — Animated Landing Page
Eight independently animated sections built with `motion/react` `whileInView` — each section reveals on scroll with staggered card animations. Theme: deep space dark (`#050510`) with violet/pink/cyan gradients.

| Section | Highlight |
|---|---|
| `HeroSection` | Split layout — left tagline, right **live looping demo chat** (4 messages auto-play with AnimatePresence transitions + typing dots) |
| `FeaturesSection` | 6 glassmorphism cards — each maps to a Section 4.1 project brief requirement with a coloured tag badge |
| `HowItWorks` | 3 animated steps with a connecting gradient line on desktop |
| `CompanionSection` | Human 👤 + AI 🤖 friendship visual, bouncing avatars, 3 example chat cards |
| `TrustSection` | 6 ethical safeguard pillars + amber disclaimer banner |
| `TestimonialSection` | **Infinite auto-scroll horizontal marquee** — 8 cards duplicated, scrolls via `motion x: ["0%", "-50%"]`, pauses on hover, left/right fade gradients |

### 2 — Chat UI (`ChatMessage.jsx`)
The entire chat experience in a single file.

**Stale-closure-free conversation tracking:**
```js
const conversationIdRef = useRef(null);
// sendMessage() always reads conversationIdRef.current
// never captures a stale closure from useState
```

**Typewriter animation:**
AI responses are typed character by character at 12ms intervals using a `setInterval` approach — can be stopped mid-stream.

**Quick-reply chips:**
5 pre-written prompts on the welcome screen. One click → calls `sendMessage(text)` directly, no typing needed.

**Copy + Text-to-Speech:**
Every AI message has two action buttons beneath it:
- Copy to clipboard
- Read aloud via Web Speech API (`SpeechSynthesisUtterance`, rate 0.9)

**Stop generation:**
The Send button becomes a red Stop button while the AI is responding. Clicking it aborts the fetch and commits whatever partial typewriter text exists to the message list.

### 3 — Find a Professional
A dedicated button in the chat top bar opens a full-screen modal with 6 dummy mental health professionals (name, speciality, city, availability, phone). Each card has a "Contact" button.

### 4 — Mock Web-Call Modal
Clicking Contact on a professional opens a full-screen **simulated voice call UI**:
- **Phase 1 — Calling (0–3s):** 3 concentric pulsing rings radiate from the avatar in the professional's colour palette
- **Phase 2 — Connected:** rings stop, live call timer counts up, avatar glows
- **Controls:** Mute toggle (turns red), Speaker toggle (turns red), large red End Call button (phone icon rotated 135°)
- Background has a radial colour wash matching the professional's gradient

### 5 — Sidebar (`Sidebar.tsx`)
Collapsible conversation sidebar:
- Search bar filters conversation titles
- Active conversation highlighted in violet
- Hover on any chat item → red X delete button appears
- "Clear all history" button with a confirmation step
- Collapses to icon-only strip (hamburger shows when fully hidden)

### 6 — Admin Dashboard
Protected by `role === "admin"`:
- `AdminDashboard.jsx` — user list, conversation counts
- `AdminStats.jsx` — LLM call logs, token usage, rate limit hits
- `ProfileDrawer.jsx` — slide-out panel with user profile details

---

## Landing Page — Section 4.1 Coverage

Every mandatory feature from the project brief is represented visually:

| Brief requirement | Landing section | Component |
|---|---|---|
| Basic conversational support | "Conversational Support" card | `FeaturesSection.tsx` |
| Stress detection patterns | "Stress Detection" card (tag: Smart Detection) | `FeaturesSection.tsx` |
| Suggest resources / counselor contact | "Resource Suggestions" card (tag: Crisis Support) | `FeaturesSection.tsx` |
| Safe responses | "Safe Responses Only" card (tag: Ethical Safeguard) | `FeaturesSection.tsx` |
| Ethical safeguards (mandatory) | "Ethical by Design" card + full Trust section | `FeaturesSection.tsx` + `TrustSection.tsx` |
| Disclaimers | Amber banner at bottom of Trust section | `TrustSection.tsx` |
| Privacy protection | "100% Private" card + Trust section pillars | `FeaturesSection.tsx` + `TrustSection.tsx` |

---

## Key Libraries

| Library | Version | Used for |
|---|---|---|
| `react` | 19 | UI framework |
| `react-router-dom` | 7 | Client-side routing |
| `motion/react` | 12 | All animations |
| `@ant-design/icons` | latest | Icon set throughout |
| `react-markdown` | latest | Render AI markdown responses |
| `tailwindcss` | 4 | All styling (utility-first) |
| `antd` | latest | Admin UI components, dark theme |

---

## Backend Connection

The backend URL is set once at the top of `ChatMessage.jsx`:

```js
const API_BASE_URL = "http://localhost:4433/api";
```

For production, move this to a Vite env variable:
```js
const API_BASE_URL = import.meta.env.VITE_API_URL;
```
and add `VITE_API_URL=https://your-api.com/api` to `FE/.env`.
