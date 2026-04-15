# BUILD_PLAN.md — ShipMosquito in 2–4 weekends

This is a concrete, ordered task list. Each task is sized to be one session in Claude Code. Strike them off as you go.

**Assumptions:**
- Next.js 15 + shadcn/ui template already set up
- You have accounts: Supabase, Anthropic, Dodo Payments, Resend, Vercel
- ~12 hours per weekend available

---

## Pre-flight (60 minutes, do before weekend 1)

- [ ] Buy domain `shipmosquito.com` (Namecheap or Cloudflare, ~$12)
- [ ] Create Supabase project (free tier)
- [ ] Create Anthropic API key, set $20 spending cap
- [ ] Create Dodo Payments account; switch to test mode; create "ShipMosquito Pro" subscription product at $29/mo (note the `product_id`)
- [ ] Create Resend account, verify domain
- [ ] Create Vercel project, link to GitHub repo
- [ ] Copy `PRD.md` and `CLAUDE.md` into repo root
- [ ] Set up `.env.local` with all keys from CLAUDE.md
- [ ] Push initial commit

---

## Weekend 1 — Core generation loop (12 hours)

**Goal:** A logged-in user can describe what they shipped and see 5 drafts.

### Saturday morning (3h) — Auth + DB foundations
- [ ] Set up Supabase clients in `lib/supabase/client.ts` and `lib/supabase/server.ts`
- [ ] Create database migration with `profiles`, `projects`, `generations`, `drafts` tables (see PRD §7)
- [ ] Enable RLS on all tables, add policies (user can only access their own rows)
- [ ] Build `/login` page with magic link + GitHub OAuth via Supabase Auth UI or custom shadcn form
- [ ] Build auth middleware that redirects unauthed users away from `/(app)/*`
- [ ] Profile creation trigger: on `auth.users` insert, create `profiles` row

### Saturday afternoon (3h) — Project management
- [ ] Build `/(app)/page.tsx` dashboard: list user's projects, "+ New project" CTA
- [ ] Build `/(app)/projects/new/page.tsx` form: name, URL, one-liner, audience chips, tone chips
- [ ] Server action: `createProject(formData)`
- [ ] Build `/(app)/projects/[id]/page.tsx`: project detail with generation history (empty state for now)

### Sunday morning (4h) — The generation engine 🔥
- [ ] Write `lib/anthropic.ts`: thin wrapper around Anthropic SDK with retries + cost logging
- [ ] Write `lib/prompts/reddit.ts` — encodes Reddit cultural rules (see PRD §5.1). Returns object: `{ subreddit, title, body, link_strategy }`
- [ ] Write `lib/prompts/hackernews.ts` — Show HN format. Returns `{ title, first_comment }`
- [ ] Write `lib/prompts/indiehackers.ts` — milestone format. Returns `{ title, body }`
- [ ] Write `lib/prompts/twitter.ts` — thread format. Returns `{ tweets: string[] }`
- [ ] Write `lib/prompts/email.ts` — customer email. Returns `{ subject, body }`
- [ ] Write `lib/prompts/index.ts` orchestrator: takes `{ project, shippedWhat }`, fans out to all 5 prompts in parallel, returns combined result
- [ ] Create `lib/data/subreddits.json` with the 25 seeded subreddits (see PRD §9)
- [ ] Reddit prompt accepts top-3 matched subreddits as context

### Sunday afternoon (2h) — The "What did you ship?" flow
- [ ] Build `/(app)/projects/[id]/new/page.tsx` form: textarea + audience override + tone override
- [ ] Server action: `generateDrafts(projectId, formData)` — creates `generations` row, calls orchestrator, persists 5 `drafts` rows, redirects to draft page
- [ ] Add loading state with rotating messages ("Asking Reddit what's cool today…", "Polishing your Show HN…", etc.)
- [ ] Track free-tier usage: increment `profiles.drafts_used_this_month`, block at 5

**Weekend 1 ship check:** You can log in, create a project, describe a fake shipping event, and see 5 drafts in the database. The drafts page itself can be ugly — that's weekend 2.

---

## Weekend 2 — Review UX + Payments + Launch readiness (12 hours)

**Goal:** Mobile-perfect draft review, Dodo Payments integration, public-launch-ready landing page.

### Saturday morning (4h) — Draft review page (the product)
- [ ] Build `/(app)/drafts/[id]/page.tsx`: 5 cards, one per platform
- [ ] Each card shows platform icon, title (where applicable), full content, character count
- [ ] Each card has: editable textarea (auto-saves on blur), "Copy" button, "Mark as posted" toggle, "Regenerate" button
- [ ] Reddit card additionally shows recommended subreddit + rules summary + "Open subreddit" link
- [ ] HN card shows the title separately and the first-comment separately
- [ ] **Test on iPhone Safari before moving on.** Cards must be tappable, copy must work, no horizontal scroll.

### Saturday afternoon (3h) — Email notifications
- [ ] Write `lib/resend.ts` wrapper
- [ ] Build email template "Your buzz is ready 🦟" with a deep link to the drafts page
- [ ] Trigger email at the end of `generateDrafts` server action
- [ ] Test deliverability (check spam folder)

### Sunday morning (3h) — Payments (Dodo)
- [ ] Install adapter: `pnpm add @dodopayments/nextjs dodopayments standardwebhooks`
- [ ] Write `lib/dodo.ts`: instantiate `DodoPayments` client; helper `createOrGetCustomer(profile)` that calls `dodo.customers.create` and saves `dodo_customer_id` to profile
- [ ] Build `/api/checkout/route.ts`: uses `Checkout` route handler from `@dodopayments/nextjs` OR manually creates a subscription with `payment_link: true` and redirects to the returned URL
- [ ] Build `/api/portal/route.ts`: redirects to Dodo customer portal session (uses `CustomerPortal` route handler)
- [ ] Build `/api/webhooks/dodo/route.ts`: uses `Webhook` route handler from `@dodopayments/nextjs` (signature verification built in via `standardwebhooks`); handle `subscription.active`, `subscription.cancelled`, `subscription.payment_succeeded`, `subscription.payment_failed`; update `profiles.plan` and `profiles.dodo_subscription_id`
- [ ] Add webhook endpoint URL in Dodo dashboard → Settings → Webhooks → save `DODO_PAYMENTS_WEBHOOK_KEY` to env
- [ ] Build `/(app)/settings/billing/page.tsx`: shows current plan, "Upgrade to Pro" (calls `/api/checkout`) or "Manage subscription" (calls `/api/portal`)
- [ ] Add paywall component: shows when free user hits 5/month limit, links to checkout

### Sunday afternoon (2h) — Landing page + launch prep
- [ ] Build `/(marketing)/page.tsx` with shadcn:
  - Hero: tagline "You ship. It buzzes." + 30-second loom demo
  - 3-section feature grid (input → drafts → post)
  - Pricing table (free vs Pro)
  - 3 example draft screenshots (Reddit, HN, IH)
  - FAQ (8 questions)
  - CTA: "Buzz my next ship — free"
- [ ] Add OG image (use `og-image` library or static PNG)
- [ ] Set up Plausible or Vercel Analytics
- [ ] Deploy to production at `shipmosquito.com`
- [ ] Test full flow end-to-end on production with a Dodo test card (see Dodo docs for card numbers)

**Weekend 2 ship check:** A real friend who isn't you can sign up cold, generate drafts, pay $29 (with a test card), and post one draft to Reddit. If they can't do this without help, fix the gaps before public launch.

---

## Weekend 3 (optional) — GitHub trigger + Telegram (12 hours)

**Goal:** The dogfooding killer feature. Push a release → drafts appear on your phone.

### Saturday (6h) — GitHub webhook
- [ ] Add `github_repo` and `github_webhook_secret` to projects
- [ ] Build `/(app)/projects/[id]/integrations/page.tsx` to connect a GitHub repo
- [ ] Use GitHub App or instruct user to add webhook manually (App is cleaner)
- [ ] Build `/api/webhooks/github/route.ts`: verifies signature, parses release event, creates generation, runs orchestrator, sends notification

### Sunday (6h) — Telegram bot
- [ ] Create Telegram bot via BotFather
- [ ] Add `telegram_chat_id` to profiles
- [ ] Build connection flow: user clicks "Connect Telegram" → opens t.me link → bot stores chat_id
- [ ] On generation complete, send Telegram message with deep link to drafts page
- [ ] Test the magic moment: push a GitHub release, get a Telegram ping in <60 seconds

**Weekend 3 ship check:** You push a release for ShipMosquito itself. Within 60 seconds, you get a Telegram ping. You open the link on your phone, review the Reddit draft, copy it, post it. **You just dogfooded the killer feature.**

---

## Weekend 4 (optional) — Subreddit picker + history + polish (12 hours)

- [ ] Better subreddit picker UI: show top 3 with rules, member count, "best for" tags
- [ ] Draft history page: filter by project, by platform, by posted/not-posted
- [ ] Add 25 more subreddits to the curated database (50 total)
- [ ] Add Sentry for error tracking
- [ ] Add Dodo annual plan ($290/year) as a second product in the Dodo dashboard
- [ ] Write 3 SEO landing pages: "PersonaBox alternative", "Reddit post generator for SaaS", "How to write Show HN posts"
- [ ] Add a roadmap page showing what's next (lets users see momentum)

---

## Launch checklist (after v1 ships)

### Technical
- [ ] Privacy policy + terms of service (use Termly or similar, ~30 min)
- [ ] Cookie banner if targeting EU users
- [ ] Error tracking confirmed working (trigger a test error)
- [ ] Dodo Payments switched from `test_mode` to `live_mode` (`DODO_PAYMENTS_ENVIRONMENT`), webhook key rotated, production webhook URL added in Dodo dashboard
- [ ] Domain has proper email DNS (SPF, DKIM via Resend)
- [ ] OG image renders correctly on Twitter/Reddit/HN preview

### Marketing assets
- [ ] 30-second demo Loom (mobile screen recording showing the full flow)
- [ ] 3 screenshot examples of generated drafts
- [ ] Twitter / X account `@shipmosquito` registered
- [ ] IndieHackers profile created with product page

### Pre-launch (week before)
- [ ] DM 20 indie devs personally with private beta invite
- [ ] Spend 30min/day commenting helpfully on r/SideProject, r/SaaS to build karma (need 200+)
- [ ] Submit to free directories: Uneed (paid $30 fast-track is worth it), DevHunt, BetaList, MicroLaunch, Peerlist, SaaSHub, AlternativeTo
- [ ] Schedule launch sequence per PRD §12

### Launch day
- [ ] r/SideProject post (use ShipMosquito-generated draft)
- [ ] r/SaaS weekly thread post
- [ ] IndieHackers product page + journey post
- [ ] X thread (drafted by ShipMosquito)
- [ ] Show HN (next day, Tuesday/Wednesday 9am PT)
- [ ] Personal email to your network
- [ ] Reply to every comment within 30 minutes for first 6 hours

---

## Definition of done for v1

✅ A real indie dev (not you) can:
1. Sign up cold without instructions
2. Generate drafts in <20 seconds
3. Review and post a draft on their phone
4. Hit the paywall and successfully pay $29
5. Come back next week and use it again

When 3 different real users do this without help, you've shipped v1. Now go get to 35 of them.

---

## Things that will tempt you to over-build (resist)

- "Should I add LinkedIn drafts?" — No. PersonaBox owns that lane. Our wedge is dev communities.
- "Should I auto-post?" — No. Trust is the product. Defer to v2.
- "Should I support multiple LLMs?" — No. Claude only. Reduce surface area.
- "Should I add team accounts?" — No. Solo devs only. Multi-user is v2 or v3.
- "Should I track post engagement (upvotes, etc)?" — No. Requires API access we're avoiding. v2.
- "Should the prompts be user-customizable?" — No. The prompts ARE the product. Don't let users water them down.

Ship the boring, narrow, opinionated v1. Let real usage tell you what to add.
