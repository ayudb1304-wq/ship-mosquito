# CLAUDE.md

> This file is auto-loaded by Claude Code. It tells Claude what this project is, how it's structured, and what conventions to follow. Keep it under 200 lines.

## Project: ShipMosquito

**Tagline:** You ship. It buzzes.

**What it is:** A web app that turns developer shipping activity (commits, releases, feature updates) into community-tuned marketing drafts for Reddit, Hacker News, IndieHackers, X/Twitter, and customer email.

**Who it's for:** Indie developers and technical founders who ship 2–4 updates/month but skip marketing because writing for 5 platforms takes 45 minutes they don't have.

**The wedge:** Every other tool in this space (PersonaBox, ShipPost, generic AI writers) covers LinkedIn/Twitter and produces generic marketing slop. ShipMosquito covers the developer communities (Reddit, HN, IH) that actually drive indie dev first users — with prompts that encode each platform's 2026 cultural rules.

**6-month goal:** $1,000 MRR (35 paying customers at $29/mo).

See `PRD.md` for full product spec. See `BUILD_PLAN.md` for weekend-by-weekend tasks.

---

## Tech stack

- **Framework:** Next.js 15 (App Router), TypeScript, React Server Components where possible
- **UI:** shadcn/ui + Tailwind CSS (template already set up)
- **Auth + DB:** Supabase (Postgres + Auth, RLS for security)
- **LLM:** Anthropic API, model `claude-sonnet-4-6` (do NOT use opus — cost matters)
- **Payments:** Dodo Payments via `@dodopayments/nextjs` adapter (Merchant of Record — they handle tax/compliance globally)
- **Email:** Resend
- **Hosting:** Vercel
- **Error tracking:** Sentry (free tier, add in week 2)

**Deliberately NOT in stack:**
- ❌ Twitter/X API (paid tier $100/mo; we use copy-paste)
- ❌ Reddit API in v1 (use hand-curated subreddit JSON)
- ❌ Any ORM (use Supabase JS client + raw SQL where needed)
- ❌ tRPC (overkill; use server actions)
- ❌ Redux / Zustand (server components + URL state are enough)

---

## Repo structure (target)

```
/app
  /(marketing)
    page.tsx                  # landing page
    pricing/page.tsx
  /(auth)
    login/page.tsx
  /(app)
    layout.tsx                # auth-gated, sidebar
    page.tsx                  # dashboard
    projects/
      new/page.tsx
      [id]/
        page.tsx              # project detail + history
        new/page.tsx          # "what did you ship?" form
    drafts/[id]/page.tsx      # 5-platform draft review
    settings/
      page.tsx
      billing/page.tsx
  /api
    webhooks/
      github/route.ts
      dodo/route.ts
    checkout/route.ts           # creates Dodo checkout session
    portal/route.ts             # redirects to Dodo customer portal
/components
  /ui                         # shadcn primitives (already there)
  /draft-card.tsx             # the platform draft cards
  /project-form.tsx
  /generate-form.tsx
/lib
  /prompts
    reddit.ts                 # platform-culture prompts (the moat)
    hackernews.ts
    indiehackers.ts
    twitter.ts
    email.ts
    index.ts                  # orchestrator
  /data
    subreddits.json           # curated v1 dataset
  /supabase
    client.ts                 # browser client
    server.ts                 # server client
  /anthropic.ts               # Claude API wrapper
  /dodo.ts                    # Dodo Payments client + helpers
  /resend.ts
/types
  /database.ts                # generated from Supabase
  /drafts.ts                  # platform-specific draft shapes
```

---

## Code conventions

1. **Server actions over API routes** for anything internal. Reserve `/api/*` for webhooks and external callers.
2. **Server components by default.** Mark `'use client'` only when needed (forms, interactivity).
3. **Database access through `lib/supabase/server.ts`** in server contexts; never import the service-role key into client code.
4. **All Claude calls go through `lib/anthropic.ts`** — single place to handle retries, logging, cost tracking.
5. **Platform prompts in `lib/prompts/*.ts`** are the most important code in the repo. Treat them like product features. Comment them heavily. Version them.
6. **No `any` types.** Use `unknown` and narrow.
7. **Tailwind only.** No CSS modules, no styled-components.
8. **shadcn components only for UI primitives.** Don't pull in another component library.

---

## Environment variables

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=        # server-only, never expose

ANTHROPIC_API_KEY=

DODO_PAYMENTS_API_KEY=
DODO_PAYMENTS_WEBHOOK_KEY=
DODO_PAYMENTS_ENVIRONMENT=test_mode    # 'test_mode' | 'live_mode'
DODO_PAYMENTS_RETURN_URL=http://localhost:3000/checkout/success
DODO_PRO_PRODUCT_ID=                   # from Dodo dashboard

RESEND_API_KEY=
RESEND_FROM_EMAIL=hello@shipmosquito.com

GITHUB_WEBHOOK_SECRET=             # weekend 3
TELEGRAM_BOT_TOKEN=                # weekend 3 (stretch)

NEXT_PUBLIC_APP_URL=http://localhost:3000
```

Add new env vars to both `.env.local` AND this file.

---

## What "done" looks like for v1

A user can:
1. Sign up with GitHub OAuth
2. Create a project (name, URL, one-liner, audience)
3. Click "+ New buzz", describe what they shipped
4. See 5 platform drafts within 20 seconds
5. Edit, copy, mark as posted on their phone
6. Hit a paywall at 5 generations/month (free) or pay $29/mo via Dodo Payments hosted checkout (Pro)
7. Receive an email when generation completes

**v1 ships when:** A real indie dev (not the builder) can sign up cold, generate drafts, and post one to Reddit without help. Test this with 3 real users before public launch.

---

## What NOT to build (push back if asked)

- ❌ Brand voice training (v2)
- ❌ Auto-posting to any platform (trust > convenience)
- ❌ Image generation (PersonaBox's lane)
- ❌ Analytics dashboard (engagement tracking)
- ❌ Team accounts
- ❌ LinkedIn drafts (PersonaBox owns this; our wedge is dev communities)
- ❌ A/B testing of drafts
- ❌ Multi-language

If a feature isn't in PRD.md section 4 (must-have or should-have), it's out of scope for v1.

---

## When working on this codebase

- **Read `PRD.md` before adding features.** Verify it's in scope.
- **Read `lib/prompts/*.ts` before changing platform prompt logic.** These are the product.
- **Run `pnpm build` before claiming a task is done.** Catch type errors early.
- **Test mobile viewport.** This product is reviewed on phones.
- **Never auto-post.** Even when it would be easy. Trust is the product.
