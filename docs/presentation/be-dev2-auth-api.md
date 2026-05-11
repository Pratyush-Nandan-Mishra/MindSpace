# MindSpace — My Contribution: Auth, API, Database & Security
### Backend Developer 2

---

## What I worked on

I built the **foundation** of the MindSpace backend — everything that keeps the server running securely and reliably. This includes the authentication system, the database design, all the API routes, security middleware, and the admin features.

While my teammate focused on making the AI smart, I made sure the application is **secure**, **well-structured**, and **production-ready**. Without this layer, the AI pipeline has nothing to sit on.

---

## Technologies I used

| Technology | Why I used it |
|---|---|
| Node.js 22 (ES Modules) | Modern JavaScript runtime, `import/export` syntax throughout |
| Express.js | Lightweight and flexible HTTP server framework |
| MongoDB Atlas | Cloud-hosted NoSQL database — flexible schema, easy to scale |
| Mongoose ODM | Schema validation and query helpers on top of MongoDB |
| Google OAuth 2.0 | Sign-in with Google — no passwords to store or manage |
| Passport.js | Standard Node.js OAuth middleware, handles the OAuth flow |
| express-session + connect-mongo | Server-side sessions stored in MongoDB |
| JWT (jsonwebtoken) | Access tokens + refresh tokens for stateless auth verification |
| express-validator | Input validation and sanitization on all incoming requests |

---

## Feature 1: Google OAuth Authentication

I chose Google OAuth instead of building a username/password system for two reasons: security (we never store passwords) and user experience (one click to log in).

**How the flow works:**

```
1. User clicks "Get Started Free"
   → GET /api/auth/google
   → Passport.js redirects to Google's OAuth page

2. User approves → Google redirects back to:
   → GET /api/auth/google/callback
   → Passport.js receives the authorization code
   → Exchanges it for a Google profile (name, email, picture, googleId)

3. Backend checks MongoDB: does this googleId exist?
   → Yes: load the existing User document
   → No: create a new User document (first-time sign-up)

4. Session is created and stored in MongoDB (via connect-mongo)
5. JWT access token + refresh token are issued

6. Frontend stores nothing sensitive — the session cookie handles auth
```

**Key files:** `config/passport.js`, `controllers/auth.controller.js`, `routes/auth.routes.js`

---

## Feature 2: Session Management + JWT Dual-Token System

I implemented a **hybrid auth system** — sessions for the browser, JWTs for the API.

**Sessions (connect-mongo):**
- When a user logs in, Express creates a session and stores it in the MongoDB `sessions` collection via `connect-mongo`
- The browser gets a `connect.sid` session cookie (HttpOnly, so JavaScript cannot read it)
- Every subsequent request from the browser automatically sends this cookie

**JWT (access + refresh tokens):**
- Short-lived access token (used to verify identity quickly)
- Longer-lived refresh token (used to get a new access token when the old one expires)
- Implemented in `helpers/jwt.helper.js` using the `jsonwebtoken` library

**JWT middleware** (`middleware/jwt.middleware.js`):
- Validates the token on every protected route
- Rejects requests with expired or tampered tokens

---

## Feature 3: Database Design — 8 Mongoose Models

I designed all 8 data models. Each one has a specific purpose:

| Model | What it stores | Key fields |
|---|---|---|
| `User` | Every person who has logged in | `googleId`, `email`, `role` (user/admin), `picture` |
| `Conversation` | Each chat session | `userId`, array of `messages` with role + content + timestamp, `title` |
| `ChatMemory` | Gemini-generated summaries for long-term memory | `userId`, array of `summaries` |
| `FAQ` | Cached knowledge base | `question`, `answer` — text-indexed for search |
| `MessageRateLimit` | Rate limit tracking per user | `userId`, `count`, `windowStart` timestamp |
| `LLMCall` | Log of every AI API call | `userId`, `model`, `inputTokens`, `outputTokens`, `createdAt` |
| `Feedback` | User feedback submissions | `userId`, `message`, `rating` |
| `Billing` | User plan tier | `userId`, `plan` — ready for future monetization |

**Schema decisions I made:**
- `Conversation.messages` is an embedded array (not a separate collection) because messages are always read together with their conversation — embedding avoids extra joins
- `FAQ` has a MongoDB text index on both `question` and `answer` — this is what powers the AI's FAQ search
- `User.role` defaults to `"user"` — admin access is manually assigned in the database

---

## Feature 4: REST API Routes

I designed and implemented all the API routes, organized into logical groups:

**Auth routes (`/api/auth`):**
```
GET  /api/auth/google           → Kick off Google OAuth
GET  /api/auth/google/callback  → Handle OAuth return + set session
GET  /api/auth/me               → Return current user object
GET  /api/auth/logout           → Destroy session
```

**Chat routes (`/api/chat`):**
```
POST   /api/chat/chat                    → Send a message (main AI endpoint)
GET    /api/chat/conversations           → List all conversations for this user
GET    /api/chat/conversation/:id        → Load one conversation (all messages)
DELETE /api/chat/conversation/:id        → Delete a single conversation
DELETE /api/chat/conversations/clear     → Delete all conversations for this user
```

**Admin routes (admin role only):**
```
GET    /api/admin/users    → List all users + stats
GET    /api/stats          → LLM usage logs, token counts, rate limit hits
GET    /api/faq            → List all FAQ entries
POST   /api/faq            → Add a new FAQ entry
DELETE /api/faq/:id        → Remove a FAQ entry
```

---

## Feature 5: Role-Based Access Control (RBAC)

I implemented a two-tier access system.

- All logged-in routes are protected by `authMiddleware` — checks the session, rejects unauthenticated requests
- Admin routes have an additional `roleMiddleware` check — if `user.role !== "admin"`, the request is rejected with a 403

This means even if a regular user discovers the admin route URLs, they cannot access any data.

**Files:** `middleware/auth.middleware.js`, `middleware/role.middleware.js`

---

## Feature 6: Per-User Rate Limiting

I built a custom rate limiter backed by MongoDB (not in-memory, so it survives server restarts).

**How it works:**
- Every time a user sends a chat message, `ratelimit.middleware.js` looks up their `MessageRateLimit` document
- Increments the count for the current time window
- If count exceeds the limit → returns HTTP 429 with a custom message that the frontend displays in the chat as a bot response
- The time window resets automatically after the configured period

This prevents abuse and keeps API costs under control.

---

## Feature 7: Input Validation

Every route that accepts user input has a dedicated validator in `validators/`. I used `express-validator` to:
- Check that required fields are present
- Sanitize strings (trim whitespace, escape HTML)
- Validate types and lengths
- Return descriptive 400 errors when validation fails

This stops bad data from ever reaching the database or the AI pipeline.

---

## Feature 8: Utility Layer

I built three utility classes used throughout the backend:

- `utils/ApiError.js` — standardized error object with status code and message
- `utils/ApiResponse.js` — standardized success response wrapper (so every endpoint returns the same JSON shape)
- `utils/AsyncHandler.js` — wraps async route handlers so Express catches unhandled promise rejections automatically (no try/catch needed in every controller)

---

## Key Files I Owned

```
BE/src/
├── server.js                       Entry point — wires everything together
├── config/
│   ├── database.js                 MongoDB Atlas connection
│   └── passport.js                 Google OAuth strategy
├── models/                         All 8 Mongoose schemas
│   ├── User.js, Conversation.js, ChatMemory.js
│   ├── FAQ.js, MessageRateLimit.js, LLMCall.js
│   ├── Feedback.js, Billing.js
├── controllers/                    Thin route handlers
├── middleware/
│   ├── auth.middleware.js          Session-based auth guard
│   ├── jwt.middleware.js           JWT token validator
│   ├── role.middleware.js          Admin role check
│   ├── ratelimit.middleware.js     Per-user DB-backed rate limiter
│   └── validation.middleware.js    express-validator error handler
├── validators/                     Input validation rule sets
├── routes/                         Express routers
└── utils/
    ├── ApiError.js
    ├── ApiResponse.js
    └── AsyncHandler.js
```

---

## Summary

My work makes MindSpace trustworthy. Authentication, authorization, input validation, rate limiting, and a clean API structure are not visible features — but they are what separates a student demo from a production-ready application. Every request that reaches the AI engine has already been authenticated, validated, and rate-checked because of the middleware layers I built.
