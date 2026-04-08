# Medif — MediNotes Pro

**Portfolio full-stack demo:** an AI-assisted workflow for turning raw **consultation notes** into structured output—**clinical-style summary**, **provider next steps**, and a **patient-friendly email draft**—with **real-time streaming** in the browser, **authenticated APIs**, and a path to **subscription-gated** access.

In the app, the product surface is branded **MediNotes Pro** (landing + consultation assistant). The repository is **Medif**.

---

## For recruiters (scan this first)

| What you’re looking for | Where this repo shows it |
|-------------------------|---------------------------|
| **Product thinking** | Guided form (patient, visit date, free-text notes) → one coherent AI artifact with three clear sections |
| **Full-stack TypeScript + Python** | Next.js 16 / React 19 UI; FastAPI service with Pydantic models and JWT verification |
| **Streaming UX** | Server-Sent Events from FastAPI; client uses `@microsoft/fetch-event-source` + **React Markdown** for live rendering |
| **Auth in depth** | Clerk on the frontend; `fastapi-clerk-auth` + JWKS on the backend; bearer token on every generation request |
| **Monetization / gating** | Clerk `Protect` + `PricingTable` so the assistant is behind a plan |
| **Polyglot deploy** | Vercel **experimental services**: Next.js at `/`, FastAPI mounted under `/api` ([`vercel.json`](vercel.json)) |

**One sentence:** I built a small but **end-to-end** slice—UI, secure API, LLM streaming, and hosting patterns you’d use on a real SaaS—without hiding the hard parts (auth across runtimes, streaming protocol, separate env per service).

---

## What the product does

1. **Landing** ([`pages/index.tsx`](pages/index.tsx)) — value prop: summaries, action items, patient communications; Clerk sign-in; CTA into the app.
2. **Assistant** ([`pages/product.tsx`](pages/product.tsx)) — authenticated users submit **patient name**, **date of visit**, and **consultation notes**; the UI streams the model response as markdown.
3. **API** ([`api/index.py`](api/index.py)) — `POST` accepts JSON (`patient_name`, `date_of_visit`, `notes`), verifies the Clerk JWT, calls an OpenAI-compatible API with a fixed **system prompt** (three markdown sections), and streams tokens as SSE.

**Important:** This is a **demo / portfolio** project. It is **not** a certified medical device, and **you must not** use it for real PHI without proper legal, security, and compliance review. UI copy about compliance is illustrative only.

---

## Architecture

```text
Browser (Next.js)
  ├── Marketing + Clerk (/)
  └── /product: form → POST /api + Bearer JWT
                        │
                        ▼
              FastAPI (Python)
              • Clerk JWT (JWKS)
              • OpenAI-compatible client (e.g. OpenRouter)
              • SSE stream → markdown sections
```

---

## Tech stack

| Layer | Choices |
|-------|---------|
| **Frontend** | Next.js 16 (Pages Router), React 19, TypeScript, Tailwind CSS 4, `@tailwindcss/typography` |
| **Forms / UX** | `react-datepicker`, controlled inputs, loading states during stream |
| **Auth & plans** | `@clerk/nextjs`, `Protect`, `PricingTable` |
| **Streaming & content** | `@microsoft/fetch-event-source`, `react-markdown`, `remark-gfm`, `remark-breaks` |
| **Backend** | FastAPI, Pydantic, `StreamingResponse`, `fastapi-clerk-auth`, `openai` SDK |

---

## Prerequisites

- Node.js 20+
- Python 3.12+ (for local FastAPI)
- [Clerk](https://clerk.com) app (frontend keys + JWKS URL for the API)
- OpenAI-compatible provider (e.g. [OpenRouter](https://openrouter.ai)) — API key and base URL

---

## Environment variables

**Next.js (Clerk)** — e.g. `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`, plus any domain/settings Clerk documents for your Next.js version.

**FastAPI** (`api/index.py`):

| Variable | Purpose |
|----------|---------|
| `OPENROUTER_API_KEY` | API key for the OpenAI-compatible provider |
| `OPENROUTER_BASE_URL` | Base URL for that provider |
| `CLERK_JWKS_URL` | Clerk JWKS URL for `Authorization: Bearer` verification |

Set these for **both** the frontend and Python service on Vercel.

---

## Local development

**Frontend**

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

**Backend** (full stack with real auth and streaming)

```bash
pip install -r requirements.txt
# export OPENROUTER_API_KEY, OPENROUTER_BASE_URL, CLERK_JWKS_URL
uvicorn api.index:app --reload --port 8000
```

In development, `/api` on the Next host matches production only if you proxy to FastAPI or use your provider’s split setup; otherwise point the client at your local API URL while testing.

**Production build (frontend)**

```bash
npm run build && npm start
```

---

## Deploy

Target: **[Vercel](https://vercel.com)** with `experimentalServices` in [`vercel.json`](vercel.json): Next.js serves the site; FastAPI is the `/api` backend. Configure environment variables for **each** service.

---

## Project structure (notable paths)

| Path | Role |
|------|------|
| `pages/index.tsx` | Landing — MediNotes Pro, feature grid, Clerk CTAs |
| `pages/product.tsx` | Plan gate + consultation form + SSE markdown output |
| `pages/_app.tsx` | `ClerkProvider` |
| `pages/_document.tsx` | Default document title / meta |
| `api/index.py` | Authenticated streaming consultation endpoint |
| `vercel.json` | Frontend / backend service split |

---

## License

MIT
