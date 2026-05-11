# MindSpace — My Contribution: Chat App, Dashboard & Auth
### Frontend Developer 2

---

## What I worked on

Once a user clicks "Get Started" and logs in, everything they interact with is what I built. I was responsible for the **core application** — the chat interface, the sidebar, the user dashboard, the admin panel, and the authentication flow that ties it all together.

While my teammate handled the public landing page, I handled the **private, logged-in side** of MindSpace.

---

## Technologies I used

| Technology | Why I used it |
|---|---|
| React 19 | All app pages are React components |
| React Router DOM v7 | Client-side routing between pages |
| React Context API | Global auth state shared across the whole app |
| Tailwind CSS v4 | Consistent utility-first styling |
| Motion v12 | Smooth UI transitions inside the chat |
| Ant Design + dark theme | Ready-made UI components for the admin panel |
| react-markdown | Renders the AI's formatted responses correctly |
| Web Speech API | Built-in browser API for text-to-speech |
| Native `fetch` | All API calls to the backend |

---

## Auth Flow — How Login Works

I built the complete authentication experience on the frontend.

```
1. User clicks "Get Started Free" on the landing page
   → Redirected to backend Google OAuth
   → Google consent screen appears
   → Backend sets a session cookie and returns the user

2. App loads → AuthContext calls GET /api/auth/me
   → Gets back { name, email, role, picture }
   → Stores it in React state, available everywhere

3. Every API call automatically sends the session cookie
   → credentials: "include" on every fetch() call
```

I built `AuthContext.jsx` which wraps the entire app. Any component in the app can call `useAuth()` to get the current user without prop drilling.

I also built `ProtectedRoute.jsx` — a route guard that:
- Redirects unauthenticated users to `/auth`
- Redirects users with the wrong role (e.g., a regular user trying to reach `/admin/dashboard`) to the home page

---

## Routing — Page Structure

| URL | What it shows | Who can access |
|---|---|---|
| `/` | Landing page | Anyone |
| `/auth` | Google login button | Anyone (redirects away if already logged in) |
| `/dashboard` | User's chat interface | Logged-in users |
| `/admin/dashboard` | Admin user management | Admin role only |
| `/admin/stats` | Admin analytics | Admin role only |
| `*` (anything else) | Falls back to landing | Anyone |

---

## Chat UI — The Core Feature

The chat interface (`ChatMessage.jsx`) is the most complex component I built. It handles everything:

### Message sending and receiving
- User types a message → `POST /api/chat/chat` is called → AI response streams back
- While waiting, a **typing indicator** (animated dots) shows in the chat
- The AI response renders with **typewriter animation** — each character appears one at a time at 12ms intervals, making it feel like the AI is actually thinking and typing

### Stop generation
- While the AI is responding, the Send button turns into a red **Stop button**
- Clicking Stop cancels the fetch request immediately and locks in whatever partial text the typewriter had rendered so far
- The partial response is saved to the message list so nothing is lost

### Quick-reply chips
- On the welcome screen (before the first message), 5 pre-written prompts are shown as clickable chips
- One click sends that message directly — no typing needed
- Helpful for users who aren't sure what to say first

### AI message actions
- Every AI response has two buttons underneath it:
  - **Copy** — copies the message text to clipboard
  - **Read aloud** — uses the browser's built-in Web Speech API to read the response out loud at a slightly slower pace (rate 0.9) for clarity

### Markdown rendering
- AI responses often include bullet points, bold text, or headers
- I used `react-markdown` so these render properly instead of showing as raw symbols

### Stale-closure prevention
- A subtle but important engineering choice: I used `useRef` to track the active conversation ID instead of `useState`
- This ensures that when `sendMessage()` runs asynchronously, it always reads the *current* conversation ID — not a stale value captured when the component first rendered

---

## Sidebar

The sidebar shows all past conversations and lets users switch between them.

- **Search bar** at the top — filters conversation titles in real time
- Active conversation is highlighted in violet
- Hovering over any conversation reveals a **red X delete button**
- **"Clear all history"** button — asks for confirmation before deleting everything
- The sidebar can **collapse** down to an icon-only strip to give more screen space to the chat
- When fully hidden, a hamburger icon appears to bring it back

---

## Find a Professional

A button in the chat top bar opens a full-screen modal listing 6 mental health professionals with their name, specialty, city, availability status, and phone number. Each has a "Contact" button.

---

## Mock Web-Call Modal

Clicking "Contact" on a professional opens a simulated voice call screen I built:

- **Phase 1 (0–3 seconds):** 3 concentric rings pulse and radiate outward from the professional's avatar — the "calling" state
- **Phase 2:** Rings stop, a live call timer starts counting up, the avatar glows
- **Controls on the call screen:**
  - Mute button — turns red when muted
  - Speaker button — turns red when off
  - End Call button — large red button, phone icon rotated 135°
- The background has a radial color gradient matching that specific professional's color

This feature makes the app feel complete and shows the "connect to a real professional" pathway required by the project brief.

---

## Dashboard & Admin Panel

### User Dashboard
- Shows the user's profile picture, name, and email
- Entry point into the chat interface
- Profile drawer slides out with account details

### Admin Dashboard (admin role only)
- List of all registered users with their conversation counts
- Real-time data pulled from the backend

### Admin Stats Page (admin role only)
- LLM call logs — which model was used, how many tokens, when
- Rate limit hit counts
- Usage trends

---

## Key Files I Owned

```
FE/src/
├── context/AuthContext.jsx         Global auth state
├── components/
│   ├── Sidebar.tsx                 Collapsible conversation sidebar
│   ├── NavBar.jsx                  Top navigation bar
│   ├── ProtectedRoute.jsx          Role-based route guard
│   └── Footer.jsx
├── hooks/use-mobile.tsx            Responsive breakpoint hook
├── infra/DarkThemeProvider         Ant Design dark theme wrapper
└── pages/
    ├── Auth.jsx                    Login page
    ├── Home.jsx                    Redirect placeholder
    ├── dashboard/
    │   ├── Dashboard.jsx
    │   ├── AdminDashboard.jsx
    │   ├── AdminStats.jsx
    │   ├── Header.jsx
    │   └── ProfileDrawer.jsx
    └── features/ChatMessage.jsx    ★ Main chat UI (most complex file)
```

---

## Summary

My work covers everything the user actually interacts with after logging in. The chat UI, the typewriter effect, the stop button, the sidebar, the admin tools — these are all features I built from scratch. The biggest challenge was the chat interface itself, which required careful state management to handle async responses, streaming text, and real-time UI updates without bugs.
