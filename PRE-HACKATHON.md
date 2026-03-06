# PRE-HACKATHON.md

Everything built and decided before the hackathon kickoff (Fri March 6, 1 PM EST).

---

## The Setup

This is a solo build: Collin + Claude Code. The entire pre-hackathon period was run
as a series of focused sessions using the TANSTAAFL session system — short, scoped
work with explicit save points and full context carry-forward between sessions.

No teammates. No wasted context. Full resumability.

---

## Stack Decisions (Session S036 — 2026-03-04)

Stack locked before any building began:

| Tool | Role | Decision notes |
|------|------|----------------|
| Next.js + shadcn/ui + Tailwind | Frontend + API routes | Standard modern stack; Vercel-native |
| Vercel | Hosting + serverless | Free tier; auto-deploy from GitHub |
| Supabase | Database + auth | Free tier; Postgres + auth out of the box |
| Claude API (claude-sonnet-4-6) | Coaching brain | Best reasoning model for adaptive coaching |
| Gemini Nano Banana 2 | Asset generation | AI Studio API key (free tier, 500 img/day — separate from Google AI Pro subscription) |
| Loom | Demo video | Required by hackathon submission |
| GitHub (two repos) | Code + docs | Public; `flowcoach-app` + `flowcoach-docs` |

**Rejected:** Figma Make — CORS blocks Claude API calls from browser-only tools.

---

## Session P1 — 2026-03-05

**Theme:** Accounts + keys + scaffold + deploy

### Accounts Created

| Service | Account | Notes |
|---------|---------|-------|
| Vercel | cbt-dsotm (GitHub auth) | Hobby tier; GitHub App installed for flowcoach-app only |
| Supabase | cbt-dsotm (GitHub auth) | Org: cbt-dsotm; project setup deferred to P2 |
| Loom | desertwaveai@gmail.com | Workspace: desertwaveai |

### API Keys

| Key | Location | Notes |
|-----|----------|-------|
| Anthropic API (`flowcoach-hackathon`) | Proton Pass | Verified live; rotated once after accidental exposure |
| Google AI Studio | Proton Pass (under desertwaveai@gmail.com) | Default project key; AI Studio API free tier (500 img/day Nano Banana 2); Google AI Pro subscription does not lift this limit |

### Repos Created

| Repo | URL | Notes |
|------|-----|-------|
| flowcoach-app | github.com/cbt-dsotm/flowcoach-app | Public; app code |
| flowcoach-docs | github.com/cbt-dsotm/flowcoach-docs | Public; this repo |

### Scaffold

- Next.js 16 initialized with TypeScript, Tailwind v4, ESLint, App Router
- shadcn/ui initialized with defaults
- Pushed to `flowcoach-app` → connected to Vercel → auto-deploy confirmed

### Live URL

`https://flowcoach-app.vercel.app` — blank Next.js default page. Stack is wired.

### Environment Variables (Vercel)

- `ANTHROPIC_API_KEY` — set in Vercel dashboard for production

---

## Key Decisions Made Pre-Hackathon

**Parallel session strategy** — identify tasks that can run as clean background sessions
(docs, Hat prompts, isolated screens) and note opportunities at P1 startup. Validate
approach through P1–P3 before hackathon starts.

**Two-repo structure** — app code and build documentation are separate public repos.
Judges can read the full narrative without wading through source code.

**Pre/during doc split** — PRE-HACKATHON.md (this file) captures everything before
kickoff. PROCESS.md captures Day 1–3 only. Different audiences; different cadences.

**No assets before kickoff** — logo, icons, illustrations wait until Day 1. Keeps
the pre/during split clean. Asset generation (Nano Banana 2) is a Day 1 task.

**TANSTAAFL session system as competitive edge** — zero context loss between sessions.
Every decision documented. Every session picks up exactly where the last one left off.
This is what lets a solo builder move at team speed.
