# IdeaGen Pro

**AI-powered SaaS demo** that generates structured business ideas for the AI agent economy. The app pairs a polished Next.js marketing and product experience with a **Python FastAPI** backend that streams model output in real time—deployed as a **split frontend/backend** project on Vercel.

---

## Why this project (for recruiters)

This repo is a small but **end-to-end product slice**: authentication, subscription gating, a streaming LLM integration, and a **polyglot** deployment (TypeScript/React + Python) rather than a single monolithic API route. It shows comfort with modern frontend tooling, secure API design, and platform-native hosting patterns.

| Area | What it demonstrates |
|------|----------------------|
| **Frontend** | Next.js 16, React 19, TypeScript, Tailwind CSS 4, responsive landing + in-app UI |
| **Auth** | Clerk (sign-in, session, user menu) |
| **Monetization path** | Clerk `Protect` + `PricingTable` for plan-gated features |
| **Backend** | FastAPI, JWT verification via `fastapi-clerk-auth`, user-scoped requests |
| **AI** | OpenAI-compatible client (OpenRouter), **server-sent events (SSE)** streaming |
| **Client ↔ API** | `@microsoft/fetch-event-source` consuming the stream; **React Markdown** for rich output |
| **Deploy** | Vercel **experimental services**: Next.js at `/`, FastAPI mounted under `/api` |

---

## Architecture (high level)

```text
Browser (Next.js)
  ├── Marketing + auth UI (/)
  └── Product page (/product): Clerk JWT → SSE to /api
                                    │
                                    ▼
                          FastAPI (Python)
                          • Verifies Clerk JWT
                          • Streams completion from OpenRouter
```

The Python API lives in [`api/index.py`](api/index.py). The product UI streams tokens into markdown in [`pages/product.tsx`](pages/product.tsx). Routing is configured in [`vercel.json`](vercel.json).

---

## Tech stack

- **Frameworks:** Next.js (Pages Router), FastAPI  
- **Languages:** TypeScript, Python  
- **UI:** React 19, Tailwind CSS 4, `@tailwindcss/typography`  
- **Auth & billing (Clerk):** `@clerk/nextjs`, subscription protection + pricing UI  
- **AI:** `openai` SDK against OpenRouter; streaming responses  
- **Markdown:** `react-markdown`, `remark-gfm`, `remark-breaks`

---

## Prerequisites

- Node.js 20+ (aligns with current Next.js expectations)
- Python 3.12+ (for local FastAPI)
- [Clerk](https://clerk.com) application (JWT templates / JWKS for the backend)
- [OpenRouter](https://openrouter.ai) (or compatible) API key and base URL

---

## Environment variables

**Next.js (Clerk)** — use the keys from your Clerk dashboard (e.g. `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`, and any publishable domain settings Clerk documents for your Next.js version).

**FastAPI (`api/index.py`)** — the backend expects:

| Variable | Purpose |
|----------|---------|
| `OPENROUTER_API_KEY` | API key for the OpenAI-compatible provider |
| `OPENROUTER_BASE_URL` | Base URL for that provider (e.g. OpenRouter) |
| `CLERK_JWKS_URL` | Clerk JWKS URL used to verify `Authorization: Bearer` tokens |

Configure these in Vercel for the Python service and locally via `.env` or your shell when running uvicorn.

---

## Local development

**Frontend**

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

**Backend (optional, for full SSE against real auth)**

Install Python dependencies and run FastAPI (from the repo root):

```bash
pip install -r requirements.txt
# Set OPENROUTER_*, CLERK_JWKS_URL, then e.g.:
uvicorn api.index:app --reload --port 8000
```

For the **same origin** as in production (`/api` on the Next host), rely on Vercel’s split services or configure your dev proxy so `/api` forwards to the FastAPI port—otherwise point `fetchEventSource` at your local API URL during experiments.

**Production build (frontend)**

```bash
npm run build && npm start
```

---

## Deploy

This project targets **[Vercel](https://vercel.com)** with `experimentalServices` in [`vercel.json`](vercel.json): Next.js serves the site; FastAPI is deployed as the `/api` backend. Set environment variables for **both** the frontend and the Python service in the Vercel project settings.

---

## Project structure (notable paths)

| Path | Role |
|------|------|
| `pages/index.tsx` | Landing: hero, pricing preview, Clerk CTAs |
| `pages/product.tsx` | Gated idea generator + SSE + markdown |
| `pages/_app.tsx` | `ClerkProvider` wrapper |
| `api/index.py` | Authenticated streaming idea endpoint |
| `vercel.json` | Frontend/backend service split |

---

## License

Private / portfolio use unless you add an explicit license.
