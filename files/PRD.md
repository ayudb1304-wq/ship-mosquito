# ShipMosquito — Product Requirements Document

> **You ship. It buzzes.**
>
> The ship-to-share engine that turns every commit, release, and feature into community-tuned marketing posts for Reddit, Hacker News, IndieHackers, and X — drafted while you're at your day job, ready to review on your phone in 3 minutes.

---

## 0. At a glance

| Field | Value |
|---|---|
| **Product name** | ShipMosquito |
| **Primary tagline** | You ship. It buzzes. |
| **Backup taglines** | • The little buzz behind every ship.<br>• Stop letting your launches die in silence.<br>• Small bites. Big distribution. |
| **One-liner** | ShipMosquito turns what you ship into what gets shared — with platform-tuned drafts for Reddit, HN, IndieHackers, and X delivered to your phone. |
| **Target user** | Indie developers and technical founders who ship 2–4 updates/month but skip marketing because writing for 5 platforms takes 45 minutes they don't have. |
| **Pricing** | Free tier (5 drafts/month) → $29/mo Pro (unlimited drafts, GitHub auto-trigger, all platforms) |
| **North-star metric** | Drafts shipped to a real platform per active user per week (target: 3+) |
| **6-month goal** | $1,000 MRR (35 paying customers) |
| **Build budget** | <$20/month infra, 2–4 weekends to v1 |
| **Stack** | Next.js 15 (App Router), shadcn/ui, Supabase (Postgres + Auth), Anthropic API (Claude Sonnet 4.6), Resend, **Dodo Payments** (Merchant of Record), Vercel |

---

## 1. The problem

Indie developers are 10× better at building than at distribution. The specific pain isn't "I don't know marketing exists" — it's the friction at the moment of shipping:

- It's 11 PM. They just pushed a feature. They know they *should* tell people.
- Writing 5 versions (Reddit post that won't get removed, HN Show post with the right tone, IndieHackers update with metrics, X thread, customer email) takes 45+ minutes.
- Each platform has different cultural rules. HN hates superlatives. Reddit hates promo. IH expects transparency.
- They open ChatGPT, get generic slop, give up, post nothing. Feature ships into silence.

**The pattern repeats every release.** Output piles up. Audience doesn't.

### Why existing tools fail this user

| Tool | Price | Why it fails |
|---|---|---|
| **PersonaBox** | $49+/mo | LinkedIn + X only. No Reddit, HN, IH. GitHub PR-only input. Priced for B2B teams. |
| **ShipPost** | Free (BYOK) | Just a Claude wrapper. No platform culture, no triggers, no review workflow. |
| **Buffer / Typefully** | $5–19/mo | Schedulers. Don't generate content. Don't understand dev communities. |
| **Generic AI writers** | Varies | Produce surface-level marketing slop devs reject by day 3. |
| **ReplyAgent** | Varies | Spammy auto-poster. Opposite of what authentic devs want. |

**The wedge:** Platform-culture intelligence for the developer communities (Reddit, HN, IH) that actually drive indie dev first users — at solo-dev pricing — with human-in-the-loop review on mobile.

---

## 2. Product principles

1. **Drafts, not posts.** v1 never auto-posts. Trust is earned by giving the user the last word.
2. **Mobile review is first-class.** The user reviews on their phone during their commute. Desktop is for setup.
3. **Platform culture is the moat.** Anyone can call Claude. Few can encode "what doesn't get downvoted in r/SaaS in 2026."
4. **Dogfood ruthlessly.** Every ShipMosquito feature ships through ShipMosquito. The growth chart is the demo.
5. **Boring stack, fast ship.** Next.js + Supabase + shadcn. Nothing fancy. Ship in weekends.

---

## 3. Target user

### Primary persona — "Sam the Shipping Solo Dev"
- 28–40 years old, software engineer at a tech company
- Has 1–3 side projects live; one earning $0–500 MRR
- Ships 2–4 updates/month after work hours
- Has $50–100/mo discretionary tool budget
- Comfortable with GitHub, terminal, deploying to Vercel
- **Hates:** generic "AI marketing assistant" tools, having to think about marketing, getting downvoted, posting cringe
- **Wants:** more users, validation that the work matters, some path to quitting the day job

### Secondary persona — "Maya the OSS Maintainer"
- Maintains an open-source project with 500–10,000 GitHub stars
- Wants to grow stars, contributors, and (eventually) revenue from a hosted version
- Ships releases monthly
- Same pain: writing release-announcement content for 5 platforms is exhausting

### Anti-persona (do NOT build for)
- Marketing teams at funded startups (PersonaBox already serves them; they need approval workflows, brand guidelines, analytics dashboards)
- Non-technical solopreneurs (LinkedIn-first audience; PersonaBox/Taplio fits better)
- Agencies managing many client accounts (multi-tenant complexity not worth it for v1)

---

## 4. v1 scope (the 2-weekend cut)

### Must-have (v1)

**Input**
- Manual "What did you ship?" form with fields:
  - Product name
  - Product URL
  - One-liner about the product (saved per project)
  - What did you ship? (free-text, 1–3 paragraphs)
  - Target audience (chips: developers, indie hackers, designers, marketers, other)
  - Tone preference (chips: technical, founder-story, casual, formal)

**Generation**
- Single button "Generate drafts" calls Claude with platform-specific prompts
- Generates 5 drafts in parallel:
  1. **Reddit** — recommended subreddit + title + body + first-comment link strategy
  2. **Show HN** — title + first comment (the "story" comment that lands the post)
  3. **IndieHackers** — milestone-style update with transparency framing
  4. **X / Twitter thread** — 4–7 tweet thread, hook-first
  5. **Customer email** — short update for existing users

**Review**
- Drafts page with one card per platform
- Each card: editable text area, "Copy" button, "Mark as posted" button
- Mobile-responsive (this is non-negotiable; test on iPhone before shipping)
- "Regenerate this one" button per platform with optional steering note

**Notifications**
- Email via Resend when drafts are ready ("Your buzz is ready 🦟")
- Optional: Telegram bot integration (deep link to draft) — *stretch for weekend 2*

**Auth + Payments**
- Supabase Auth (email magic link + GitHub OAuth)
- Dodo Payments hosted checkout: $29/mo Pro plan
- Dodo Payments customer portal for subscription management
- Free tier: 5 draft generations/month (1 generation = 1 set of 5 platform drafts)
- Pro tier: unlimited generations + GitHub trigger
- **Dodo as Merchant of Record** handles VAT/GST/sales tax globally — zero tax compliance work for the founder

**Project model**
- A user has 1+ Projects
- A Project stores: name, URL, one-liner, default audience, default tone
- Eliminates re-entering context every time

### Should-have (weekend 3 if scope holds)

- **GitHub release webhook trigger** — connect a repo, ShipMosquito auto-generates drafts on every release published. This is the dogfooding killer feature.
- **Subreddit picker UI** — show top 3 subreddit recommendations with rules summary, member count, and "best post type" hints
- **Draft history** — see past drafts and which were marked as posted

### Won't-have (v1)

- ❌ Auto-posting to any platform (trust + API costs + risk)
- ❌ X / Twitter API integration (paid tier is $100/mo; copy-paste works fine)
- ❌ Brand voice training (PersonaBox does this; defer to v2)
- ❌ Analytics dashboard (engagement tracking)
- ❌ Team accounts / multi-user
- ❌ Native mobile app (responsive web is enough)
- ❌ Image generation (PersonaBox's hook; we win on platform fit, not visuals)
- ❌ Scheduling (we generate; user posts when ready)

---

## 5. The platform-culture prompts (the actual moat)

This is the core IP. Each platform gets a prompt that encodes its 2026 cultural rules. **These prompts live in `/lib/prompts/` and should be treated as the most important code in the repo.**

### 5.1 Reddit prompt rules
- Lead with the problem the user had, not the product
- Title format: "I built X because Y was broken" or "Here's what I learned shipping X"
- Body: 200–400 words, includes a vulnerability or a metric
- NO em-dashes (the AI tell that gets posts removed in 2026)
- NO "game-changer," "revolutionize," "unleash"
- Recommend posting link in first comment, not body
- Match recommended subreddit's flair conventions

### 5.2 Show HN prompt rules
- Title format: `Show HN: ProductName – plain description of what it does`
- No marketing language in the title. Period.
- First comment is critical: explains motivation, technical decisions, what's hard, what's open-source
- Acknowledge limitations openly
- Mention the stack
- 150–300 word first comment

### 5.3 IndieHackers prompt rules
- Lead with a number (revenue, users, time, something concrete)
- Story arc: where I was → what I shipped → what happened → what I'm doing next
- Explicitly invite questions/feedback at the end
- 300–500 words

### 5.4 X / Twitter thread rules
- Hook tweet must work standalone (no "thread 🧵" only)
- 4–7 tweets total
- One idea per tweet
- Last tweet has the link
- No hashtag spam (max 1)

### 5.5 Customer email rules
- Subject < 50 chars, no emoji in subject
- 100–200 words body
- One clear CTA
- Sign off as a human, not a company

---

## 6. Information architecture

```
/                       Marketing landing page
/login                  Magic link + GitHub OAuth
/app                    Dashboard (project list)
/app/projects/new       Create project
/app/projects/[id]      Project detail (settings + draft history)
/app/projects/[id]/new  "What did you ship?" form
/app/drafts/[id]        Draft review page (5 platform cards)
/app/settings           Account, billing, integrations
/app/settings/billing   Dodo customer portal redirect
/api/webhooks/github    GitHub release webhook
/api/webhooks/dodo      Dodo Payments events
/api/checkout           Creates Dodo checkout session
/api/portal             Redirects to Dodo customer portal
/api/generate           Server action that calls Claude
```

---

## 7. Data model (Supabase / Postgres)

```sql
-- Users come from Supabase Auth (auth.users)

create table profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  email text not null,
  plan text not null default 'free',           -- 'free' | 'pro'
  dodo_customer_id text,
  dodo_subscription_id text,
  drafts_used_this_month int not null default 0,
  drafts_reset_at timestamptz not null default now(),
  created_at timestamptz not null default now()
);

create table projects (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references profiles(id) on delete cascade,
  name text not null,
  url text,
  one_liner text,
  default_audience text,
  default_tone text,
  github_repo text,                            -- 'owner/repo' if connected
  github_webhook_secret text,
  created_at timestamptz not null default now()
);

create table generations (
  id uuid primary key default gen_random_uuid(),
  project_id uuid not null references projects(id) on delete cascade,
  user_id uuid not null references profiles(id) on delete cascade,
  shipped_what text not null,                  -- raw input from user or webhook
  audience text,
  tone text,
  source text not null default 'manual',       -- 'manual' | 'github_release'
  created_at timestamptz not null default now()
);

create table drafts (
  id uuid primary key default gen_random_uuid(),
  generation_id uuid not null references generations(id) on delete cascade,
  platform text not null,                      -- 'reddit' | 'hackernews' | 'indiehackers' | 'twitter' | 'email'
  content jsonb not null,                      -- platform-specific structure
  edited_content jsonb,                        -- user's edits
  posted_at timestamptz,                       -- user marks as posted
  created_at timestamptz not null default now()
);

-- RLS policies: user can only see their own rows
```

---

## 8. Tech stack & dependencies

| Layer | Choice | Why |
|---|---|---|
| Framework | Next.js 15 (App Router) | shadcn template assumed; server actions for Claude calls |
| UI | shadcn/ui + Tailwind | User has template ready |
| Auth | Supabase Auth | Free, magic link + GitHub OAuth out of box |
| DB | Supabase Postgres | Free tier 500MB; RLS for security |
| LLM | Anthropic API (claude-sonnet-4-6) | Best quality/cost; user already uses Claude |
| Payments | Dodo Payments + `@dodopayments/nextjs` adapter | Merchant of Record handles tax/compliance globally; cleaner DX than Stripe; better for solo founders selling internationally |
| Email | Resend | 3K/month free, dev-friendly API |
| Hosting | Vercel | Free tier handles early traffic |
| Analytics | Plausible (paid, $9/mo) OR Vercel Analytics (free) | Privacy-friendly |
| Error tracking | Sentry (free tier) | Catch generation failures |
| Telegram (stretch) | grammY (Node lib) | Free, deep links work great |

**No paid APIs in v1.** Twitter API explicitly avoided. Reddit API explicitly avoided (use a hand-curated subreddit database for v1).

---

## 9. The curated subreddit database (v1 ship-with content)

Build a static JSON file at `/lib/data/subreddits.json` with ~30 subreddits. Each entry:

```json
{
  "name": "r/SaaS",
  "members": 264000,
  "audience_tags": ["developers", "indie hackers", "founders"],
  "rules_summary": "Self-promo allowed in weekly thread (Sundays). Direct posts must offer value first. No 'check out my product' titles.",
  "best_post_format": "I built X because Y. Here's what I learned: [3 specific lessons]. [Link in comments].",
  "post_link_in": "comments",
  "good_for": ["launches", "milestones", "lessons-learned"],
  "avoid_if": ["pure feature announcement", "no story angle"]
}
```

Seed list (start with these 25):
- r/SideProject, r/SaaS, r/indiehackers, r/microsaas, r/startups, r/EntrepreneurRideAlong
- r/webdev, r/programming, r/learnprogramming, r/javascript, r/reactjs, r/nextjs, r/node, r/Python, r/golang, r/rust
- r/selfhosted, r/opensource, r/devops, r/MachineLearning, r/LocalLLaMA
- r/productivity, r/ChatGPTCoding, r/cursor, r/ClaudeAI

The Generate prompt receives the relevant subreddit entries as context based on the project's `default_audience` chips.

---

## 10. UX flows

### Flow 1: First-time user (the wow moment)
1. Land on `/` → tagline + 30-second demo video → "Buzz my next ship" CTA
2. Sign up with GitHub OAuth (one click)
3. Onboarding modal: "What are you building?" (auto-creates first project)
4. Land in dashboard with one example draft pre-generated using their project info ⭐ **(this is the wow)**
5. Show them what good looks like before they spend any effort

### Flow 2: Manual generation (the daily flow)
1. From dashboard → "+ New buzz" → form on phone or desktop
2. Paste/type what shipped → tap Generate
3. ~15 second loading state with rotating messages ("Asking Reddit what's cool today…")
4. Land on draft review page with 5 cards
5. Tap a card → edit inline → Copy → switch to Reddit app → paste → post
6. Back in ShipMosquito → tap "Mark as posted"

### Flow 3: GitHub-triggered (the magic flow, weekend 3)
1. Settings → Connect GitHub → pick repo → ShipMosquito installs webhook
2. User pushes a release on GitHub
3. Within 60 seconds: email + Telegram ping "Your release is ready to buzz"
4. Tap link → land on drafts page on phone → review → post
5. **Total user effort: opening the email.**

---

## 11. Pricing & monetization

| Tier | Price | Limits |
|---|---|---|
| **Free** | $0 | 5 generations/month (= 25 drafts), manual input only, all platforms |
| **Pro** | $29/mo | Unlimited generations, GitHub auto-trigger, draft history (90 days), Telegram bot, priority generation |
| **Pro annual** | $290/yr (save $58) | Same + 2 months free |

**Why $29:** Below PersonaBox ($49+), above the unsustainable $9 trap, lands inside indie dev discretionary tool budget. **35 paying users = $1,000 MRR.**

**No usage-based pricing in v1.** Predictable cost is a feature for the target user.

---

## 12. Build-in-public dogfooding plan

Every ShipMosquito feature ships through ShipMosquito. The growth chart IS the demo.

| Week | Ship | Buzz it on |
|---|---|---|
| 1 | MVP private beta | DM 20 indie devs personally |
| 2 | Public launch | r/SideProject, r/SaaS, IndieHackers (drafts generated by ShipMosquito) |
| 3 | GitHub trigger | Show HN (draft generated by ShipMosquito) |
| 4 | Telegram bot | r/indiehackers, X thread |
| 5 | Subreddit picker | Dev.to article + cross-post |
| 6 | Draft history + analytics | IndieHackers milestone post: "$X MRR after Y weeks" |

Every post includes a screenshot of the actual draft ShipMosquito generated for that post. **The recursion is the marketing.**

---

## 13. Success metrics

### Leading indicators (week 1–4)
- Signups per day
- % of signups who complete first generation (target: >70%)
- % of generations where user marks ≥1 draft as "posted" (target: >40%)

### Lagging indicators (month 2–6)
- Free → Pro conversion rate (target: >5%)
- MRR (target: $1,000 by month 6)
- Logo retention month-over-month (target: >85%)

### Qualitative
- Tweets / posts mentioning ShipMosquito unprompted
- "I posted this with ShipMosquito" disclosures in the wild

---

## 14. Risks & mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| PersonaBox adds Reddit/HN | Medium | Move fast; own the dev community wedge before they notice |
| Claude API costs spike with growth | Low | Cache project context; usage cap on free tier; sonnet not opus |
| AI-generated posts get banned from subreddits | Medium | Anti-AI-tells in prompts; user reviews before posting; never auto-post |
| Dodo Payments downtime/issues (newer than Stripe) | Low | MoR benefits outweigh; abstract behind `lib/payments/` so swapping is a 1-day job if needed |
| Twitter/X demands API even for copy-paste | Low | We never touch the API; user posts manually |
| Indie devs don't pay for marketing tools | Medium | Validated by PersonaBox at $49; we're cheaper and more relevant |
| GitHub webhook reliability | Low | Retry logic; manual re-trigger button |
| User finds drafts generic | High | Platform-culture prompts are the moat; iterate weekly based on feedback |

---

## 15. Out of scope (explicitly)

These come up but should NOT be in v1:
- LinkedIn drafts (PersonaBox owns this; we win by NOT being them)
- Image / OG card generation
- Multi-language support
- Team / agency accounts
- Bring-your-own-LLM (BYOK)
- A/B testing of drafts
- Analytics on engagement (likes, upvotes, etc.)
- Browser extension
- VS Code extension
- CLI

Park these in a `/ROADMAP.md` for v2 conversations.
