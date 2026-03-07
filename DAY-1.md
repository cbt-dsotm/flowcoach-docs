# DAY-1.md

Everything that happened on Day 1 — Friday, March 6, 2026.

Day 1 starts at the kickoff livestream (1 PM EST / 10 AM PST).
Day 1 ends when Collin goes to bed.

---

## The Day in One Sentence

Built a working sprint loop, hit a wall with architecture, tore it down, and designed something fundamentally better.

---

## What Got Built

### S1 — Screen Skeleton

Five routes, all navigable: `/` → `/onboarding` → `/goal` → `/sprint` → `/progress`. Nothing functional yet, but the full flow was clickable end-to-end. The skeleton existed before we wrote a single line of real logic — this made every subsequent session faster because the shape of the product was already visible.

### S2 — Onboarding Wizard

Six questions, one at a time. Energy level, learning style, focus pattern, feedback preference, session length, optional notes. Each answer saves to a Supabase `profiles` table keyed by session. The interface uses a progress bar across both the screen-level steps (Step 1 of 4) and the question-level steps (1 of 6) — two zoom levels of progress simultaneously.

### S3 — Goal Screen

Topic input, optional win condition, confidence rating 1–5. Saves to Supabase `goals` table. The confidence slider is a prior — it tells the coach where to start, not where to stay.

### S4 — Sprint Loop Core

The first real coaching turn. White Hat system prompt built from the learner's profile + goal. Full conversation history (last 20 messages). Hat switcher UI — all six hats visible and tappable. Difficulty buttons wired to the API. Auto-start kickoff on mount.

A specific problem we solved here: the sprint was opening mid-lesson, jumping straight into a quiz without introducing the topic. Fix: the kickoff message explicitly instructs the coach to orient first, then teach, then exercise. The sprint now opens with context before asking anything.

### S5 — Difficulty Calibration

"How does this feel?" — Too easy / Just right / Too hard. Pre-selected to "Just right" so there's always a signal, never null. Resets to "Just right" after each turn. The signal is prepended to the user's next message so Claude sees it as context, not a separate command.

---

## What Broke (and How We Fixed It)

### The Session Cookie Problem

Every API route expected a `session_id` cookie to exist. Nothing was creating it. On the MacBook Pro this was masked by a stale cookie from earlier testing — but switching to the Mac Mini (fresh browser, no cookies) exposed the bug immediately: every API call returned 400 "No session ID."

Fix: `/api/onboarding` and `/api/goal` now generate a session ID if one doesn't exist and return it as a `Set-Cookie` header. The session cookie is created server-side on the first API call, not assumed.

**What this taught us:** The Mac Mini switch was the best thing that could have happened. Fresh environment found a bug that would have silently broken the app for any new user.

### The Vercel Build Failures

Five consecutive Vercel deploys failed with `npm run build exited with 1`. The culprit: a TypeScript error in `sprint/page.tsx` — a newer version of `react-markdown` no longer accepts `className` directly on the component. The fix was wrapping the component in a `div` with the className. One line change, five failed deploys. Lesson: always verify the build passes before testing.

---

## The Architecture Rethink

By end of Day 1, the sprint loop was working — but a deeper conversation revealed that the fundamental design was wrong.

**The original design:** a chat window that changes personality when you switch hats. White Hat = factual tone, Red Hat = warm tone, Green Hat = playful tone. All in one stream.

**The problem:** switching hats mid-conversation doesn't actually feel like switching modes. It just feels like the same chatbot with a slightly different writing style. The hats aren't meaningfully different — they're cosmetic.

**The insight:** de Bono's Six Hats were designed for evaluating something from multiple angles simultaneously. In a learning context, that means the same concept explained through six different lenses — not six different chat personalities. The hats should be *tabs*, not *modes*.

### What FlowCoach Actually Is

Not a chatbot. A **learning operating system** with five distinct modes:

| Mode | Job |
|------|-----|
| **Learn** | Six Hat tabs showing a concept from different angles |
| **Practice** | Retrieval practice, Socratic method, Feynman technique |
| **Flashcards** | Spaced repetition built from learning history |
| **Quiz** | Diagnostic assessment with responsive feedback |
| **Profile** | Deep learner model — who you are, how you think |

The dashboard shows all five modes as cards. Working modes are live. Unbuilt modes say "coming soon." The judge sees the full product vision the moment they log in — not a single clever feature, but the first working piece of a learning operating system.

### The New Learn Mode

When a learner enters a topic, hat content is generated on-demand and cached. Tap White Hat — it generates a factual explanation and caches it. Tap Green Hat — it generates an analogy or creative reframe and caches it. Switch away and back: same content is there.

Difficulty control is per-hat. "Too easy" regenerates *that hat's content* one level deeper. "Too hard" regenerates it simpler. The learner has independent control over the depth of each lens. Run White Hat at expert depth and Green Hat at beginner level simultaneously. This is the Flow channel made tangible — the learner has a steering wheel and a gas pedal.

Exercises are a separate layer. Content and assessment are decoupled. Explore the hats until ready, then hit "Test yourself." The exercise evaluates the answer and recommends which hat to try next. "Your answer was close but missed a common failure mode — try Black Hat before moving on."

### The Six Hats Redesigned

| Hat | Lens | Learning Science |
|-----|------|-----------------|
| White | Facts, definitions, structure | Explicit instruction |
| Black | Common mistakes, failure modes, edge cases | Misconception-based learning |
| Yellow | Why it matters, practical relevance | Motivation / relevance |
| Green | Analogy, metaphor, creative reframe | Dual encoding |
| Red | Intuition surfacing + emotional validation | Prior knowledge elicitation |
| Blue | Where are you, what's left, what's next | Metacognition |

Red Hat does double duty: it asks "what does your gut tell you about this before we correct it?" (prior knowledge elicitation — one of the highest-leverage techniques in learning science) AND validates emotional experience when the material feels overwhelming. Both are valid. Both are educationally grounded.

---

## Where Day 1 Ends

**Live at `flowcoach-app.vercel.app`:**
- Full onboarding → goal → sprint flow
- White Hat coaching, working end-to-end
- Red and Green Hat prompts written (routing wired, not yet tested in new architecture)
- Session cookie bug fixed
- Build green

**Designed but not yet built:**
- Hat tabs (parallel content, cached)
- Per-hat difficulty regeneration
- Separate exercise layer
- Five-mode dashboard
- Auth (Supabase login/signup)
- Deep profile mode

**Build sequence for Day 2:**
1. Auth — Supabase login/signup; persistent user identity
2. Dashboard — mode cards, navigation, coming-soon stubs
3. Learn mode rebuild — hat tabs, on-demand generation, difficulty control, exercises
4. Profile mode — deep profile conversation
5. Everything else as far as we get

---

## How It Felt

Day 1 ended with more architecture than code — which is the right outcome. Building the wrong thing fast is worse than designing the right thing carefully. The sprint loop exists as proof of concept. The real product design came from watching it work and asking "but does this actually do what we said it does?" The answer was no. The redesign took two hours of conversation and zero lines of code. That's efficient.

The Mac Mini switch, the cookie bug, the five failed Vercel builds — none of that was wasted time. Each one revealed something real. The product is tighter for it.
