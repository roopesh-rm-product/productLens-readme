<<<<<<< HEAD
<div align="center">
  <img src="images\productlensLogo.png" alt="ProductLens Logo" width="120" />
  <h2>ProductLens</h2>
  <p><strong>AI-Powered Product Understanding & Decision Engine</strong></p>

  [![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi)](https://fastapi.tiangolo.com/)
  [![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)](https://reactjs.org/)
  [![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
  [![Hugging Face](https://img.shields.io/badge/Hugging%20Face-Qwen-FFD21E?style=for-the-badge)](https://huggingface.co/Qwen/Qwen2.5-72B-Instruct)
</div>

---

## 📖 Table of Contents
- [About ProductLens](#-about-productlens)
- [Key Features](#-key-features)
- [Visual Walkthrough](#-visual-walkthrough)
- [Core Architecture](#-core-architecture)
- [Project Structure](#-project-structure)
- [Tech Stack](#-tech-stack)
- [Getting Started](#-getting-started)
- [Production Deployment](#-production-deployment)
- [API Reference](#-api-reference)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [License](#-license)

---

## ✨ About ProductLens

ProductLens is an AI tool for **QA Engineers**, **Product Managers**, and **Business Analysts**. It turns product documentation into structured insights, test ideas, bug decisions, and a live requirement traceability matrix—grounded in what you upload.

Upload a PRD, spec, or Confluence export, then use four tabs:

- **Understand** — sticky document Q&A at top; collapsible sections for scope, features, flows, rules, acceptance criteria, and open questions
- **Validate** — AI-generated test scenarios, edge cases, and risks from Understand data (with regenerate); plain-language strings safe for display
- **Decide** — classify bug reports with confidence scores and searchable, paginated discussion history
- **Traceability** — deterministic BA pipeline: LLM extracts requirements only; validation is mapped with acceptance-criteria-first logic, tiered similarity, and concise summaries; then Gap → Decision → Impact, plus coverage banner, filters, and matrix chat

**Export** — Download analysis as **Excel**, **Word**, or **PDF** (entire analysis or custom tab selection) from the analysis header.

Public pages: **`/about`**, **`/about-me`** (feedback form, light/dark theme).

**Trust layer** — AI Confidence Health banner + per-requirement **source references** (section/AC labels, document excerpts, verification against uploaded text).

---

## 🚀 Key Features

### Ingestion & history
- 📂 **Multi-format input:** PDF, HTML, web link, or pasted text
- 📦 **5 MB file upload cap** enforced on both frontend and backend
- 📄 **Up to 15 pages analyzed** per document (PDF page markers or ~52k characters for pasted/HTML text); longer uploads are trimmed to the first 15 pages before AI and citation checks
- 🕒 **Search history:** Paginated list of past analyses per user (PostgreSQL)
- 🔐 **Google OAuth** with **DB-backed sessions** (sliding 1-day expiry while active)
- ⏳ **"Signing in…" loading state** and a backend keep-alive ping from the login page to soften Render free-tier cold starts

### AI confidence & trust
- 🩺 **Analysis Health banner** on analysis pages (`overall`: ok / caution / warning) with actionable messages:
  - Incomplete documentation (`is_sufficient`)
  - Open questions count
  - Low document confidence (`document_confidence` 0–100 on Understand)
  - Unclear requirements (traceability `clarity` / low `confidence` / Clarify decisions)
  - Unverified source citations (excerpt not found in uploaded text)
  - Low bug classification confidence
  - Missing traceability / many matrix gaps
- 📎 **Source referencing** on each traceability row: `source_ref` with label (e.g. *Section 4.2 → Checkout Flow*), short **excerpt** from the doc, optional page (PDFs parsed with `--- Page N ---` markers), and **Matched in document** / verify warning
- ⚠️ Disclaimer: *AI-assisted draft — verify sources and decisions before release.*

> **Note:** Re-uploading the same PRD can still produce different AI output (non-deterministic LLM). Health and citations improve **reviewability**, not bitwise repeatability.

### Export
- 📥 **Excel (.xlsx)**, **Word (.docx)**, and **PDF** — generated on the backend (`openpyxl`, `python-docx`, `reportlab`)
- 🎛 **Entire analysis** or **custom** scope (checkboxes: Understand, Validate, Decide, Traceability, Chat history)
- 📊 **Excel:** one row per list item; traceability validation/gaps in **separate cells** (extra rows per requirement when needed)
- 📄 **Word & PDF:** same section structure (summary, bullets, bug details, traceability table with expanded rows)
- ⚡ **Traceability tab shortcut:** “Export matrix” pre-selects Excel + Traceability only

### Understand tab
- 💬 **Sticky document chat** at the top — quick prompts + Q&A grounded in the doc and insights (does not scroll away); **chat history persisted** in PostgreSQL per user/document
- 📂 **Collapsible sections** — expand/collapse: documentation status, summary, scope, features, flows, rules, acceptance criteria, open questions
- 📋 **Documentation quality** banner (`is_sufficient`) + **document confidence** (0–100%)
- 🎯 **Scope:** in-scope / out-of-scope lists
- 🧩 **Key features**, **user flows**, **business rules**
- ✅ **Acceptance criteria** grouped by feature
- ❓ **Open questions** for PM/BA follow-up

### Validate tab
- 🧪 **Test scenarios**, **edge cases**, and **risk areas** — generated from **Understand** only (not raw document re-read); not blocked by documentation confidence scores
- 🔄 **Regenerate** — `POST /api/documents/{id}/regenerate-validate` rebuilds scenarios from stored Understand data
- 🛡 **Normalization** — list items are coerced to readable strings even when the model returns objects
- 📋 **Deterministic fallback** — if the LLM fails, scenarios are built from features, flows, rules, and acceptance criteria already in Understand

### Decide tab (Bug Decision Engine)
- ⚖️ **Classification:** Bug / Expected Behavior / Missing Requirement
- 📊 **Confidence score** (0–100%)
- 📝 **Reasoning**, **impact**, and **recommendation**
- 🔍 **Bug discussions:** search + paginated history (10 per page) per document

### Traceability tab (Requirement Intelligence)
- 🧭 **Step-by-step engine** (`backend/services/traceability.py`):
  1. **Requirements** — LLM extracts 5–15 atomic, testable requirements from document + Understand (retries + deterministic fallbacks)
  2. **Validation mapping** — rule-based only (no LLM): maps test coverage to each requirement
  3. **Gaps** — e.g. no validation, unclear requirement, missing requirement (from bugs)
  4. **Decisions** — Clarify / Accept from gaps + validation (documentation rows); bug rows use Decide classifications
  5. **Bug links** — similarity match to related bug discussions
- 🎯 **Layered validation mapping:**
  - **Primary:** Understand **acceptance criteria** matched to each requirement (similarity ≥ 0.65 on feature or criterion)
  - **Fallback:** Validate tab **scenarios** when primary coverage is thin — top matches by score, capped per row
  - **Tiered match:** **Strong** (≥ 0.65), **Weak** (0.45–0.65 or keyword overlap) — weak matches still count as coverage to reduce false “No validation coverage”
  - **Readable summaries:** Gherkin-style lines are shortened for the UI (e.g. “Valid QR login succeeds” instead of full Given/When/Then)
  - **Dedupe:** case-insensitive and near-duplicate collapse; max **5** validation lines per requirement
- 📊 **Coverage banner:** total requirements, % covered, items with gaps, high-impact count, and Fix/Clarify/Defer breakdown
- 🗂 **Sub-tabs:** All / Documentation / Bugs (with live counts)
- 📎 **Source references** on each card — best-effort line from uploaded text (with verification badge); separate from validation mapping
- 🏷 **Per-row confidence & clarity** — unclear / low-confidence requirements flagged on cards
- ✨ **Smart sort:** severity → decision action → source → gap count → ID
- 🎚 **Quick filters:** Only gaps · Only Fix · High impact
- 💬 **Coverage Q&A chatbot** — matrix-aware AI assistant with quick prompts; **history persisted** in DB (`chat_type=traceability`)
- 🔄 **Auto-incorporates bug discussions** when generating and chatting

### Public pages & feedback
- 📄 **`/about`** — product overview and feature cards
- 👤 **`/about-me`** — profile, Connect links (GitHub / LinkedIn), feedback form
- ✉️ **Feedback email** — `POST /api/feedback` (Resend on Render; Gmail SMTP for local dev)
- 🌓 **Light / dark theme** toggle on all main pages

### UX & deployment
- 📱 **Responsive layout** for mobile (headers, tables, tabs, chat)
- 🔗 **Stable analysis URLs** — after upload + insights, the app navigates to `/analysis/{documentId}` so **browser reload** restores the document from the API (not the empty upload screen)
- 🏷 **Tab persistence** — active tab stored in the URL (`?tab=validate`, `?tab=traceability`, etc.); reload keeps the tab you were on
- 🔄 **SPA routing** on Vercel (`frontend/vercel.json`) so `/history`, `/analysis`, and `/analysis/:id` work on refresh
- 🧬 **Auto-migration** on backend startup: adds new JSON columns (e.g. `traceability_data`) to existing Postgres tables without Alembic

### Storage note
Uploaded **files are not stored**—only extracted text and AI JSON results (Understand, Validate, Traceability, Bug discussions, **chat messages**) are saved in the database.

---

## 📸 Visual Walkthrough

<details>
<summary><b>View App Screenshots</b></summary>

### 🔓 Login & Access
*Google OAuth sign-in.*

![Login Page](images/login.png)

### 📊 Search History
*Find and reopen past analyses.*

![Search History](images/dashboard.png)

### 📤 Document Upload
*PDF, HTML, link, or text.*

![Upload Page](images/uploadPage.png)

### 💡 AI Analysis Tabs
| Understand | Validate | Decide |
| :---: | :---: | :---: |
| ![Understand](images/understandTab.png) | ![Validate](images/validateTab.png) | ![Decide](images/bugTab1.png) |

### 🧭 Traceability tab
*Requirement matrix, coverage banner, sub-tabs, filter chips, and coverage Q&A chatbot.*

</details>

---

## 🏗️ Core Architecture

```mermaid
graph TD
    A[Documentation PDF/HTML/Text] --> B[FastAPI on Render]
    B --> C[Document Parser]
    C --> D[(PostgreSQL / Supabase)]
    B --> E[Hugging Face / Groq LLM]
    E --> F[Understand + Validate JSON]
    F --> D
    B --> T[traceability.py]
    T -->|LLM: requirements only| E
    T -->|Map validation, gaps, decisions| D
    D --> G[React SPA on Vercel]
    G -->|/analysis/:id ?tab=| B
    G -->|Coverage Q&A| E
```

---

## 📂 Project Structure

```text
productLens/
├── backend/
│   ├── main.py              # FastAPI app, CORS, health, auto-migration of new JSON columns
│   ├── models.py            # User, UserSession, Document, BugDiscussion, ChatMessage
│   ├── database.py          # SQLAlchemy + Supabase
│   ├── routes/
│   │   ├── api.py           # Upload, insights, chat, bugs, traceability, export
│   │   ├── auth.py          # Google OAuth + DB sessions (/me, /logout)
│   │   └── feedback.py      # About-page feedback → email
│   └── services/
│       ├── ai.py              # Understanding, validation, bug, doc chat, traceability
│       │                      # orchestration, normalizers, bug rows
│       ├── traceability.py    # Requirement extract (LLM) + validation map + gaps +
│       │                      # decisions + bug links (deterministic steps 2–5)
│       ├── chat_messages.py   # Persist / load document & traceability chat threads
│       ├── analysis_health.py   # Trust / confidence warnings for UI banner
│       ├── source_verify.py     # Match source_ref excerpts to parsed_text
│       ├── export_document.py # Excel, Word, PDF export builders
│       ├── feedback_email.py  # Resend (Render) / SMTP (local)
│       ├── sessions.py        # Create / validate / slide / revoke user sessions
│       └── parser.py          # PDF/HTML parsing
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Tabs/                 # Understand, Validate, Decide, Traceability
│   │   │   ├── DocumentChat.jsx      # Sticky Q&A on Understand tab (persisted)
│   │   │   ├── TraceabilityChat.jsx  # Coverage Q&A (persisted)
│   │   │   ├── ExportDialog.jsx      # Export scope + format modal
│   │   │   ├── AnalysisHealthBanner.jsx
│   │   │   ├── SourceRefBlock.jsx    # Traceability source citation UI
│   │   │   ├── AboutPage.jsx / AboutMePage.jsx / FeedbackSection.jsx
│   │   │   ├── Dashboard.jsx
│   │   │   ├── AnalysisView.jsx
│   │   │   ├── LoginPage.jsx         # Signing-in spinner + backend keep-alive
│   │   │   ├── UploadSection.jsx     # 5 MB client-side cap
│   │   │   └── SearchHistory.jsx
│   │   ├── hooks/usePersistedChat.js
│   │   └── config/api.js             # VITE_API_URL, authFetch(), session helpers, warmBackend()
│   ├── vercel.json                   # SPA rewrites (when root = frontend)
│   └── .env.example
├── render.yaml                # Render backend blueprint
├── vercel.json                # Vercel build (repo root, optional)
├── readmeImages/
├── productlens_traceability_full_spec.md   # Traceability feature spec
└── PROJECT.txt                # Extended deployment & ops guide
```

---

## 💻 Tech Stack

| Layer | Technology |
|--------|------------|
| Frontend | React 19, Vite, React Router 7, React Hot Toast, `@react-oauth/google` |
| Backend | FastAPI, Uvicorn, SQLAlchemy, openpyxl, python-docx, reportlab |
| Database | PostgreSQL (Supabase recommended for production) |
| AI | Hugging Face Inference API — `Qwen/Qwen2.5-72B-Instruct` |
| Auth | Google OAuth 2.0, opaque DB sessions (`user_sessions`, sliding expiry) |
| Hosting | Vercel (frontend), Render (backend) |

---

## 🛠️ Getting Started

### Prerequisites
- Python 3.10+
- Node.js 18+
- PostgreSQL (local or Supabase)

### 1. Backend

```bash
cd backend
python -m venv venv
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate

pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

Create a `.env` file in the **project root** (loaded by `database.py` and `ai.py`):

```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/productlens
HUGGINGFACE_API_KEY=your_huggingface_token
GROQ_API_KEY=your_groq_api_key
GROQ_MODEL=llama-3.3-70b-versatile
GOOGLE_CLIENT_ID=your_google_client_id
SESSION_EXPIRE_DAYS=1
FEEDBACK_TO_EMAIL=roopesh.rm07@gmail.com
# Render production (free tier blocks SMTP — use Resend):
RESEND_API_KEY=re_xxxxxxxx
RESEND_FROM=ProductLens <onboarding@resend.dev>
# Local dev optional Gmail SMTP:
SMTP_USER=your@gmail.com
SMTP_PASSWORD=your_gmail_app_password_no_spaces
```

### 2. Frontend

```bash
cd frontend
npm install
npm run dev
```

Create `frontend/.env`:

```env
VITE_GOOGLE_CLIENT_ID=your_google_client_id
VITE_API_URL=http://localhost:8000
```

Open **http://localhost:5173**

### Google OAuth (local + production)

In [Google Cloud Console](https://console.cloud.google.com/) → Credentials → OAuth 2.0 Client:

**Authorized JavaScript origins:**
- `http://localhost:5173`
- `https://your-production-domain.com`

Use the **same** Client ID in backend `GOOGLE_CLIENT_ID` and frontend `VITE_GOOGLE_CLIENT_ID`.

---

## 🌐 Production Deployment

Typical setup:

| Service | Role |
|---------|------|
| **Vercel** | React frontend (`frontend/` as Root Directory) |
| **Render** | FastAPI backend (`backend/`) |
| **Supabase** | PostgreSQL (use **Session pooler** URL on Render) |

### Render (backend)

- **Root Directory:** `backend`
- **Build:** `pip install -r requirements.txt`
- **Start:** `uvicorn main:app --host 0.0.0.0 --port $PORT`

**Environment variables:**

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | Supabase Session pooler URI (not direct `db.*` if IPv6 fails) |
| `HUGGINGFACE_API_KEY` | Hugging Face token (primary LLM) |
| `GROQ_API_KEY` | [Groq](https://console.groq.com) API key — **automatic fallback** when HF credits/limits fail |
| `GROQ_MODEL` | Optional (default `llama-3.3-70b-versatile`) |
| `GOOGLE_CLIENT_ID` | Same as frontend |
| `SESSION_EXPIRE_DAYS` | Sliding session lifetime in days (default `1`) |
| `FEEDBACK_TO_EMAIL` | Inbox for feedback (default `roopesh.rm07@gmail.com`) |
| `RESEND_API_KEY` | **Recommended on Render** — [Resend](https://resend.com) API key (HTTP, not blocked) |
| `RESEND_FROM` | Optional sender, e.g. `ProductLens <onboarding@resend.dev>` until you verify a domain |
| `SMTP_USER` | Local dev: Gmail address (SMTP blocked on **Render free tier**) |
| `SMTP_PASSWORD` | Gmail app password — **no spaces** (16 characters) |
| `SMTP_HOST` | Optional (default `smtp.gmail.com`) |
| `SMTP_PORT` | Optional (default `587`) |
| `FRONTEND_URL` | e.g. `https://productlens.space` |

Or deploy via `render.yaml` at the repo root.

### Vercel (frontend)

- **Root Directory:** `frontend`
- **Environment variables:**
  - `VITE_API_URL` = your Render API URL (e.g. `https://your-api.onrender.com`)
  - `VITE_GOOGLE_CLIENT_ID` = Google client ID
- **Redeploy** after changing `VITE_*` variables (baked in at build time)
- Ensure `frontend/vercel.json` exists for SPA routing (`/history`, `/analysis`, etc.)
- **SEO:** `public/robots.txt` and `public/sitemap.xml` list indexable public pages only (`/`, `/login`, `/about`, `/about-me`). Authenticated routes are disallowed. `www` redirects to apex via `vercel.json`.

#### Google Search Console (productlens.space)

1. **Property:** Use the **domain** property or URL prefix `https://productlens.space` (not `http://` — those URLs redirect and show as “Page with redirect”, which is expected).
2. **Preferred host:** In Settings, prefer **non-www** (`productlens.space`); `www` 308-redirects to apex after deploy.
3. **Sitemap:** Submit `https://productlens.space/sitemap.xml`.
4. **Do not expect indexing** of `/history`, `/analysis`, or `/analysis/:id` (login-required; blocked in `robots.txt`).
5. After deploy, use **URL Inspection** → **Request indexing** for `/`, `/about`, and `/login`.
6. Validate **Page indexing** after a few days; “Page with redirect” for `http://` or old `/dashboard` URLs should drop off.

### Supabase

1. Create a project → **Database** → **Connection string** → **Session pooler**
2. Reset password if needed → paste into Render `DATABASE_URL`
3. Tables are created automatically on first API start (`create_all`)

See **PROJECT.txt** for troubleshooting (CORS, IPv6, login, 404 on refresh).

---

## 📡 API Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | API welcome |
| `GET` | `/health` | Health check (also pinged by the login page to warm the backend) |
| `POST` | `/api/auth/google` | Google login → session token + `expires_at` |
| `GET` | `/api/auth/me` | Validate session, slide expiry, return user |
| `POST` | `/api/auth/logout` | Revoke current session |
| `POST` | `/api/feedback` | Submit feedback (`message`; guests add `contact_name` + `contact_email`) — Resend on Render, SMTP locally |
| `POST` | `/api/upload` | Upload document (5 MB max; first 15 pages stored for analysis) |
| `POST` | `/api/generate-insights/{id}` | Generate Understand + Validate (includes `health`); frontend then routes to `/analysis/{id}` |
| `POST` | `/api/documents/{id}/regenerate-validate` | Regenerate Validate tab from stored Understand (no confidence gating) |
| `GET` | `/api/documents` | List documents (`skip`, `limit`, `q`) |
| `GET` | `/api/documents/{id}` | Get document insights (Understand + Validate + Traceability) + `health` trust summary |
| `DELETE` | `/api/documents/{id}` | Delete document |
| `POST` | `/api/analyze-bug` | Bug decision (+ saves discussion if authenticated) |
| `GET` | `/api/documents/{id}/bug-discussions` | List bug discussions (`skip`, `limit`, `q`) |
| `GET` | `/api/documents/{id}/chat-messages?chat_type=document\|traceability` | Load persisted chat thread for the signed-in user |
| `DELETE` | `/api/documents/{id}/chat-messages?chat_type=document\|traceability` | Clear persisted chat thread |
| `POST` | `/api/documents/{id}/chat` | Document Q&A (`message` only; history loaded from DB) |
| `POST` | `/api/documents/{id}/generate-traceability` | Generate / regenerate the traceability matrix |
| `POST` | `/api/documents/{id}/traceability-chat` | Coverage Q&A grounded in the matrix + validation + bugs (`message` only; history from DB) |
| `POST` | `/api/documents/{id}/export` | Download export file (see below) |

**Export request body** (`POST /api/documents/{id}/export`):

```json
{
  "format": "xlsx",
  "mode": "full",
  "sections": []
}
```

| Field | Values |
|-------|--------|
| `format` | `xlsx` (Excel), `docx` (Word), `pdf` |
| `mode` | `full` — all sections; `custom` — use `sections` |
| `sections` | When `mode` is `custom`: `understand`, `validate`, `bugs`, `traceability`, `chats` (at least one) |

Response: file bytes with `Content-Disposition: attachment`. Works on Render (pure Python libs; no extra ports).

**`health` object** (on `GET /api/documents/{id}` and after `generate-insights`):

```json
{
  "overall": "caution",
  "warnings": [
    {
      "code": "incomplete_document",
      "severity": "warning",
      "tab": "understand",
      "message": "Documentation appears incomplete..."
    }
  ],
  "disclaimer": "AI-assisted draft — verify sources and decisions before release."
}
```

**Traceability row fields** (after generate / normalize):

| Field | Description |
|-------|-------------|
| `source_ref.kind` | `section`, `acceptance_criteria`, `feature`, `bug`, `page` |
| `source_ref.label` | Human label, e.g. `Section 4.2 → Checkout Flow` |
| `source_ref.excerpt` | Short quote from uploaded text |
| `source_ref.source_verified` | `true` / `false` if excerpt matched `parsed_text` |
| `confidence` | 0–100 for the requirement row |
| `clarity` | `clear` or `unclear` |

Authenticated routes require header: `Authorization: Bearer <session_token>`. Each request slides `expires_at` forward by `SESSION_EXPIRE_DAYS` (default **1**). After 24h idle, the session expires and the client must sign in again.

---

## 🗺️ Roadmap

- [x] Scope, open questions, acceptance criteria (Understand)
- [x] Document chat on Understand tab
- [x] Bug discussion history with pagination
- [x] Bug decision confidence score
- [x] Sticky chat + collapsible sections on Understand tab
- [x] Validate tab crash fix (object → string normalization)
- [x] Mobile-responsive UI
- [x] 5 MB file upload limit (frontend + backend)
- [x] Signing-in spinner + backend keep-alive on login page
- [x] Traceability tab — Requirement → Validation → Gap → Decision → Impact matrix
- [x] Coverage banner, sub-tabs, smart sort, and filter chips on Traceability
- [x] Matrix-aware coverage Q&A chatbot (Traceability)
- [x] Auto-migration for new JSON columns on startup
- [x] Persist chat threads in database (per user, per document, document + traceability threads)
- [x] Export analysis (Excel / Word / PDF) — entire or custom tab selection; Excel one-item-per-cell; PDF aligned with Word
- [x] About / About me pages + feedback email
- [x] AI Confidence Health banner (document, requirements, bugs, citations)
- [x] Source referencing on traceability (label + excerpt + verification)
- [x] Traceability validation mapping — AC-first, global scenario fallback, tiered strong/weak match
- [x] Concise validation summaries (Gherkin → short labels on matrix cards)
- [x] Analysis URL persistence (`/analysis/:id` + `?tab=` after upload; reload-safe)
- [ ] Deterministic re-runs / versioned analysis snapshots
- [ ] Confluence API direct integration
- [ ] Multi-document knowledge search
- [ ] Team collaboration spaces

---

## 🤝 Contributing

Contributions are welcome! For major changes, please open an issue first.

1. Fork the repo
2. Create a feature branch
3. Commit your changes
4. Open a Pull Request

---

## 📄 License

This project is proprietary and confidential.

Unauthorized copying, redistribution, modification, or commercial usage of any part of this repository is strictly prohibited.

Access is provided only for authorized contribution purposes.

---

<div align="center">
  <p>Made with ❤️ for the QA & Product Community</p>
  <p><sub>Designed and Developed by Roopesh Ramakanth</sub></p>
</div>
=======
# productLens-readme
Readme file for productlens repo.
>>>>>>> a1665611a576063fff5176c1167ba831f3700abf
