# FlowCoach

**An AI learning companion that keeps you in Flow state.**

FlowCoach adapts to your energy and thinking style in real time — using short sprints, a difficulty feedback loop, and Edward de Bono's Six Thinking Hats as tap-to-switch coaching modes.

Built solo (Collin + Claude Code) for the [Skool Vibe Code Hackathon](https://skool.com/hackathon), March 6–8, 2026.

---

## Who It's For

Neurodivergent and self-directed learners who are smart but inconsistent. Generic AI tutors don't adapt to your state. FlowCoach does.

---

## What It Does

1. **Onboarding** — quick interview: energy level, learning style, preferences
2. **Goal-setting** — narrow a topic to a winnable 20–30 min target
3. **Sprint loop** — micro-explanation → exercise → feedback → adjust difficulty
4. **Six Hats** — tap to switch coaching mode mid-session (White, Red, Green, Yellow, Black, Blue)
5. **Flow meter** — visual signal showing whether you're in the Flow channel

---

## Stack

| Tool | Role |
|------|------|
| Next.js + shadcn/ui + Tailwind | Frontend + API routes |
| Vercel | Hosting + serverless |
| Supabase | Database + auth |
| Claude API (claude-sonnet-4-6) | Coaching brain |
| Gemini Nano Banana 2 | Asset generation |
| Loom | Demo video |

---

## Docs in This Repo

- [PRE-HACKATHON.md](./PRE-HACKATHON.md) — everything built and decided before kickoff
- PROCESS.md — how the hackathon went, session by session *(Day 1–3)*
- ARCHITECTURE.md — system design and data model *(Day 1–3)*
- PROMPTS.md — the actual system prompts for each coaching mode *(Day 2–3)*
- LESSONS-LEARNED.md — post-hackathon capture *(post-event)*

---

## Live App

[flowcoach-app.vercel.app](https://flowcoach-app.vercel.app)

## App Repo

[github.com/cbt-dsotm/flowcoach-app](https://github.com/cbt-dsotm/flowcoach-app)
