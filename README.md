KavyaResto — Project Overview

Purpose
- A full-stack restaurant ordering system (KavyaResto) with an Admin area, Customer UI, and a Node/Express backend API. Supports OTP-based signup verification and JWT authentication.

High-level architecture
- Frontend
  - Vite + React (TypeScript) app in `Frontend/`.
  - Organized into `customer`, `admin`, and `superadmin` sections. Uses `AuthContext` and `CartContext`.
  - Uses `import.meta.env.VITE_API_URL` to point to the backend base URL.
- Backend
  - Node.js + Express in `Backend/` with route groups: `/api/auth`, `/api/menu`, `/api/orders`, `/api/payments`.
  - MongoDB (Atlas) via `MONGO_URI`.
  - Email sending: prefers SendGrid, falls back to Gmail SMTP via Nodemailer, with a console mock fallback.
  - JWT for authentication.
- Deployment
  - Frontend and backend are deployed as separate services (example: Render). Root `package.json` may delegate frontend build.

Main folders & files
- Frontend/
  - `package.json` – scripts (dev/build), `.env` with `VITE_API_URL` for production.
  - `src/customer` – Login, Signup, Menu, Cart, Checkout pages.
  - `src/admin` – Admin UI pages.
  - `src/context` – `AuthContext.tsx`, `CartContext.tsx`.
- Backend/
  - `index.js` – server entrypoint: CORS, body parser, routes, health checks, request logger.
  - `controllers/authController.js` – signup, verifyOtp, resendOtp, login, getProfile; `sendOtpEmail` helper.
  - `routes/authRoutes.js` – `/signup`, `/verify-otp`, `/resend-otp`, `/login`, `/profile`.
  - `routes/internalRoutes.js` – debug/test routes like `/internal/test-email`.
  - `models/User.js` – mongoose schema for users.
  - `middleware/authMiddleware.js` – validates JWT and sets `req.userId`.

API endpoints (summary)
- POST /api/auth/signup — create/update a user, store OTP, and send OTP via email. Returns 200 on success; may return 500 if email send fails.
- POST /api/auth/verify-otp — verify OTP, set user verified, and return JWT + user.
- POST /api/auth/resend-otp — generate and email a new OTP.
- POST /api/auth/login — expects `{ email, password }`, requires verified user, returns `{ token, user }` on success.
- GET /api/auth/profile — protected route returning user profile.
- POST /internal/test-email — debug endpoint to exercise email helper and return provider details.

Environment variables
- MONGO_URI — MongoDB connection string.
- JWT_SECRET — JWT signing secret.
- SENDGRID_API_KEY — SendGrid API key (optional).
- EMAIL_USER — SMTP user (Gmail) for fallback (optional).
- EMAIL_PASS — SMTP password or app password for Gmail fallback.
- EMAIL_FROM — default `from` address for outgoing emails.
- FRONTEND_ORIGIN / FRONTEND_PROD_ORIGIN — allowed origins for CORS allowlist.
- VITE_API_URL — frontend environment variable pointing to backend base URL.
- DEV_RETURN_OTP — when `true`, signup/resend endpoints return OTP in response for dev testing.
- DEV_DEBUG — when `true`, backend may include error stack in 500 responses (dev only).

How to run locally (developer workflow)
1. Backend
   - Copy `.env.example` to `.env` and configure `MONGO_URI`, `JWT_SECRET`, and email credentials if testing email.
   - cd Backend
   - npm install
   - npm start
2. Frontend
   - cd Frontend
   - create `.env` with `VITE_API_URL=http://localhost:5000`
   - npm install
   - npm run dev (or `npm run build` for production)

Testing OTP locally (examples)
- Signup to create user (will send OTP if email settings are valid):
  curl -i -X POST "http://localhost:5000/api/auth/signup" \
    -H "Content-Type: application/json" \
    -d '{"name":"Test User","email":"you@example.com","phone":"1234567890","password":"YourPass1!"}'

- Trigger email helper via internal test route:
  curl -i -X POST "http://localhost:5000/internal/test-email" \
    -H "Content-Type: application/json" \
    -d '{"email":"you@example.com"}'

If using Render or another host, replace `http://localhost:5000` with your deployed backend URL. If emails fail, check provider credentials and logs.

Known issues & debugging notes (from recent work)
- CORS: preflight (OPTIONS) requests sometimes failed; temporarily `app.use(cors())` was added for debugging. Replace with a strict allowlist in production.
- Render build: initial `vite: Permission denied` error fixed by invoking Vite via Node and ensuring `vite` is in dependencies.
- Startup crash: registering `app.options('*', cors())` caused path-to-regexp crash; removed such explicit options handler.
- Email sending: OTP stored but SendGrid/SMTP failing sometimes. A `POST /internal/test-email` and debug logs were added to diagnose provider errors.
- Login 500: added minimal debug logging and optional DEV_DEBUG response for diagnosing 500 responses.

Security notes
- Do not enable `DEV_DEBUG` or `DEV_RETURN_OTP` in production.
- Rotate API keys if exposed. Keep `JWT_SECRET` strong.
- Rate-limit OTP endpoints to prevent abuse.

Suggested improvements
- Harden CORS with exact allowlist and credentials settings.
- Add unit and integration tests for auth and email flows with mocks.
- Add better monitoring and alerting for email provider failures.
- Improve error response shape for consistent client handling.

How I can help next
- If you give me additional files or paths, I'll analyze them and update this document.
- I can generate a `README.md` variant for `Backend/` or create docs and commit them. I just created this file at the repo root.


---
Generated: 2025-12-09
