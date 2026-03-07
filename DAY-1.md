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

## The Auth Migration (s043)

With the architecture redesigned, the session cookie system wasn't going to hold. Anonymous sessions tied to a browser cookie meant no persistence across devices, no user identity, no real profile. We replaced the entire identity layer.

**Supabase email auth end-to-end:**
- Login/signup page (`/login`) — email + password, Supabase auth
- `proxy.ts` as the Next.js 16 auth middleware (not `middleware.ts` — that's a Next.js 15 pattern; Next.js 16 broke it silently)
- `user_id` migrated across all API routes and all Supabase tables — profiles, sessions, goals, messages
- Server-side auth client (`lib/supabase-server.ts`) — reads the session cookie and authenticates every API call
- Old landing page replaced with a redirect to `/dashboard`

The `proxy.ts` / `middleware.ts` distinction is a real gotcha. Both files exist in the repo briefly — we created `middleware.ts`, it conflicted, we deleted it and confirmed `proxy.ts` was already correct.

---

## The Learn Mode Rebuild (s043)

Learn mode went from a single chat stream to six independent, cacheable perspectives on a concept.

**Hat tabs — parallel content, not chat modes:**
Each hat tab generates its own perspective on the topic on demand. Tap a tab → content streams in. Tap away and back → cached, instant. Six independent views of the same concept.

**Per-hat difficulty control:**
Each hat has its own level: -4 (extremely simple) to +4 (expert). "Make this easier" regenerates that hat's content one step simpler. "Let's go deeper" regenerates it one step more advanced. The learner can run White Hat at expert depth and Green Hat at beginner level simultaneously. This is the Flow channel made tangible.

**Technical decisions:**
- Cache key: `${hat}:${level}` — regeneration creates a new cache entry, not an overwrite
- White Hat streams on mount — it's the anchor; learners always land in a factual starting point
- `remark-gfm` required for table rendering in ReactMarkdown (Tailwind v4 note: `@plugin` directive in CSS, no `tailwind.config.js`)
- Streaming is the UX — the cursor appearing as content generates is the magic moment; preloading would remove it

**Hat color design:**
- White: white bg + dark border (semantically correct — "White Hat" should look white)
- Yellow: yellow tint
- Black: dark gray
- Red: red
- Green: green
- Blue: blue
- Active: deep, saturated. Inactive: soft tint. White hat was accidentally rendered black-on-black — fixed.

---

## Profile Mode (s044)

The profile system is built around a single insight: a form captures declared preferences. A conversation surfaces actual cognitive patterns.

**The tier ladder:**
Profile completeness maps to coaching quality in four explicit tiers:
- **Basic** — generic coaching, same for everyone
- **Good** — Claude knows your goal and background; picks the right analogies, calibrates depth
- **Great** — Claude knows your knowledge gaps and what's blocked you; fills real holes, avoids past traps
- **Exceptional** — full picture of how you think, learn, and feel; fully adaptive across every mode

The tier ladder is visible on the profile page and on the dashboard — the "what do I get for putting in the effort" question is answered explicitly.

**Seven section conversations:**
Each section is a focused conversation with its own system prompt:
1. **Quick Profile** — 5 min, covers all dimensions at lighter depth. The fastest path to Good coaching.
2. **Your Goal** — excavates the real goal under the stated one
3. **Your Background** — maps expertise for analogy bridges
4. **Your Knowledge Map** — "explain it to me" probe to surface what's solid vs. fuzzy
5. **What's Worked (and What Hasn't)** — learning history; avoids past traps
6. **How You Like to Learn** — not learning style buckets (debunked); real cognitive preferences
7. **Your Relationship With This Subject** — emotional context; goes last because earlier conversations build trust

**The three-question opening:**
Goal sections open with three framings of the same question — pragmatic, identity, and curiosity — and invite the learner to pick the one that resonates:
1. *What are you working on or building that's making you want to learn right now?*
2. *What's the bigger version of yourself you're working toward — and what do you need to learn to get there?*
3. *What keeps pulling at your attention lately? What do you find yourself coming back to?*

**Save & Finish flow:**
After enough turns, a "Save & Finish" button appears. The system asks Claude to synthesize the conversation into a 4-6 sentence profile block, saves it to `profiles.profile_data` in Supabase, and shows a confirmation screen with the saved summary.

**Profile → system prompt injection:**
`buildRichProfileBlock()` in `lib/prompts.ts` reads the saved profile sections and injects them into every hat's system prompt. If no profile exists, it falls back to the onboarding fields. The coach gets richer context automatically as the learner completes more sections.

**Dashboard redesign:**
Profile is now the top card — visually distinct (indigo) from the learning modes below (dark). Completion count and tier tagline update dynamically. "Start here →" becomes "Continue →" once any section is done. The dashboard communicates the intended flow: set up your profile, then start learning.

---

## Where Day 1 Ends

**Live at `flowcoach-app.vercel.app`:**
- Auth — Supabase email login/signup, persistent user identity across devices
- Dashboard — Profile featured at top (indigo card, tier-aware), Learn + 3 coming-soon modes below
- Learn mode — all 6 hat tabs, streaming, per-hat difficulty -4..+4, caching, remark-gfm tables
- Profile mode — tier ladder (Basic → Good → Great → Exceptional), 7 section conversations, Save & Finish flow, rich profile injected into all coaching modes
- All hat system prompts (White, Red, Green written; Yellow, Black, Blue routing through White as fallback)
- Build green

**Still to build:**
- Practice mode (conversational retrieval + Feynman; reuses /api/coach)
- Flashcard + Quiz screens (stubs minimum, live if time)
- Yellow, Black, Blue hat prompts (currently fallback to White)
- Exercise layer (scoped as post-hackathon)
- Demo video
- Submission post

---

## How It Felt

Day 1 ended with more architecture than code — which is the right outcome. Building the wrong thing fast is worse than designing the right thing carefully. The sprint loop exists as proof of concept. The real product design came from watching it work and asking "but does this actually do what we said it does?" The answer was no. The redesign took two hours of conversation and zero lines of code. That's efficient.

The Mac Mini switch, the cookie bug, the five failed Vercel builds — none of that was wasted time. Each one revealed something real. The product is tighter for it.
