# PROMPTS_SPEC.md — The platform-culture prompts

> These prompts are the product. Anyone can call Claude. Few can encode "what doesn't get downvoted in r/SaaS in 2026." Treat the files in `lib/prompts/*.ts` as the most important code in this repo.

This document specifies what each platform prompt MUST encode. Use this as the source of truth when writing or editing the prompts in `lib/prompts/`.

---

## Shared input contract

Every platform prompt receives the same context object:

```ts
type GenerationContext = {
  project: {
    name: string;
    url?: string;
    one_liner?: string;
  };
  shipped_what: string;        // user's free-text description
  audience: string;            // 'developers' | 'indie hackers' | etc.
  tone: string;                // 'technical' | 'founder-story' | 'casual' | 'formal'
};
```

Each prompt returns a platform-specific structured object.

---

## Universal anti-AI-tell rules (apply to ALL prompts)

In 2026, these patterns get content removed from dev communities or downvoted instantly. **EVERY prompt must include these rules.**

```
DO NOT use:
- Em-dashes (—) anywhere. Use periods or commas instead.
- "It's not just X, it's Y" constructions
- "Game-changer," "revolutionize," "unleash," "empower," "leverage," "delve," "tapestry"
- "In today's fast-paced world" or any opener referencing the modern era
- Three-item lists in titles ("Fast, Simple, and Powerful")
- "Whether you're a X or a Y" constructions
- Excessive emojis (max 1 per post body)
- Bullet points unless explicitly asked for

DO write:
- Short, declarative sentences
- Specific numbers and concrete details
- One vulnerability or limitation per post (builds trust)
- Like a human who shipped code at 11pm and wants to tell people
```

---

## 1. Reddit prompt (`lib/prompts/reddit.ts`)

### Returns
```ts
type RedditDraft = {
  recommended_subreddit: string;       // e.g. 'r/SaaS'
  alternate_subreddits: string[];      // 2 backups
  title: string;                       // <300 chars, ideally <80
  body: string;                        // 200-400 words
  link_strategy: 'in_body' | 'in_first_comment' | 'in_title';
  first_comment_text?: string;         // if link_strategy == 'in_first_comment'
  rules_warnings: string[];            // things user must check before posting
};
```

### Cultural rules to encode

1. **Title format options** (pick one based on tone):
   - "I built X because Y was broken" (founder-story tone)
   - "Here's what I learned shipping X" (lessons tone)
   - "My X just hit Y users / $Z MRR. Here's what worked" (milestone tone)
   - **Never:** "Check out my new X" or "Introducing X"

2. **Body structure:**
   - Paragraph 1: The pain you had (be specific, name it)
   - Paragraph 2: What you tried before that didn't work
   - Paragraph 3: What you built and why it's different
   - Paragraph 4: One real metric or one vulnerable admission
   - Closing: A genuine question to the community

3. **Link strategy:**
   - r/SaaS, r/SideProject, r/indiehackers: link in first comment
   - r/microsaas, r/EntrepreneurRideAlong: link in body OK
   - r/programming, r/webdev: link only if the project is genuinely interesting code-wise; otherwise don't post

4. **Rules warnings to surface:**
   - "r/SaaS only allows direct promo in the Sunday weekly thread. Today is [day]."
   - "r/programming requires the post to be interesting beyond just 'I built X'."
   - "Account needs 200+ karma to post in r/SideProject without auto-removal."

5. **Subreddit picker logic:**
   - Match against `lib/data/subreddits.json` using `audience_tags`
   - Score by audience match + recent activity + permissiveness of self-promo rules
   - Return top 3

---

## 2. Hacker News prompt (`lib/prompts/hackernews.ts`)

### Returns
```ts
type HackerNewsDraft = {
  title: string;                       // strict format
  first_comment: string;               // 150-300 words, this is critical
  best_post_time_pt: string;           // e.g. "Tuesday 9:00am PT"
  warnings: string[];
};
```

### Cultural rules to encode

1. **Title format (strict):**
   ```
   Show HN: ProductName – plain factual description of what it does
   ```
   - NO marketing language. Period.
   - NO "the easiest way to X"
   - NO superlatives
   - Examples that work:
     - "Show HN: Plausible – open-source Google Analytics alternative"
     - "Show HN: Linear – issue tracking you'll actually like"
   - Examples that get flagged:
     - "Show HN: The fastest way to manage projects" ❌
     - "Show HN: Revolutionary AI tool for X" ❌

2. **First comment is critical** (this is what lands the post):
   - Open with: "Hey HN, I built this because [specific personal pain]"
   - Mention the stack (HN loves this)
   - Explain one technical decision and why
   - Acknowledge one limitation openly
   - Mention if any part is open-source
   - Invite specific feedback (not "let me know what you think")
   - End with how to reach you (email or X handle)

3. **Anti-patterns to avoid:**
   - Don't post on Sunday or Monday (low engagement)
   - Don't ask for upvotes (auto-flag)
   - Don't tell friends to upvote from their accounts (ring detection)
   - Don't post within 1 hour of another Show HN from your account
   - Don't include pricing in the first comment (post it if asked)

4. **Best time:** Tuesday or Wednesday, 9:00–10:00 AM Pacific.

---

## 3. IndieHackers prompt (`lib/prompts/indiehackers.ts`)

### Returns
```ts
type IndieHackersDraft = {
  title: string;                       // hook with a number
  body: string;                        // 300-500 words, story arc
  group: string;                       // suggested IH group
};
```

### Cultural rules to encode

1. **Title formula:**
   - Lead with a number or concrete outcome:
     - "Hit my first $X MRR after Y months of failing"
     - "How I got my first 100 users from a single Reddit post"
     - "Shipped my Yth side project. Here's what's different this time."
   - Avoid: questions ("Should I build X?"), generic announcements

2. **Body structure (story arc):**
   - Where I was (specific, with numbers)
   - What I shipped (focus on the "what changed", not features)
   - What happened (concrete results, even small ones)
   - What I'm doing next (invites engagement)
   - Open question to the community

3. **Tone:**
   - Transparency-first (mention real revenue, real failures)
   - No corporate voice
   - First-person throughout
   - One emoji max in title, none in body

4. **Length:** 300–500 words. IH readers skim — front-load the value.

---

## 4. Twitter / X prompt (`lib/prompts/twitter.ts`)

### Returns
```ts
type TwitterDraft = {
  tweets: string[];                    // 4-7 tweets
  hook_alternatives: string[];         // 2 backup opening tweets
};
```

### Cultural rules to encode

1. **Hook tweet rules (tweet 1 of N):**
   - Must work standalone (no "🧵👇" only)
   - <240 chars (leaves room for screenshots)
   - One of these patterns:
     - Specific number: "Just hit $X MRR with [project]. The 1 thing that worked:"
     - Contrarian take: "Everyone says do X. I did Y. Here's what happened:"
     - Vulnerability: "I almost shut this down last month. Here's what saved it:"
   - NO "I'm excited to announce"
   - NO "🚀" in the hook

2. **Body tweets (2 through N-1):**
   - One idea per tweet
   - Use line breaks generously
   - Numbers, not words ("3" not "three")
   - No threading words ("Next:", "Also:") — let structure speak

3. **Final tweet rules:**
   - The link goes here (not in tweet 1 — it tanks reach)
   - Soft CTA: "If you ship code regularly, check it out: [link]"
   - Optionally: "RT the first tweet if useful"

4. **Length:** 4–7 tweets. Below 4 feels thin. Above 7 loses people.

5. **Hashtags:** Max 1, only if it's a genuine community tag (#buildinpublic).

---

## 5. Customer email prompt (`lib/prompts/email.ts`)

### Returns
```ts
type EmailDraft = {
  subject: string;                     // <50 chars
  preheader: string;                   // <90 chars
  body: string;                        // 100-200 words
  cta_text: string;
  cta_url: string;
};
```

### Cultural rules to encode

1. **Subject line:**
   - <50 characters
   - No emoji in subject (kills deliverability)
   - Specific, not clever:
     - ✅ "New: GitHub auto-trigger"
     - ❌ "Big news! 🎉"

2. **Body:**
   - Open with a one-line "what's new"
   - One paragraph of "why this matters to you"
   - One clear CTA (button)
   - Sign off with a real first name, not "The Team"
   - 100–200 words MAX

3. **No marketing tropes:**
   - No "We're thrilled to announce"
   - No "We've been working hard"
   - No "We heard your feedback" (unless quoting specific feedback)

---

## Orchestrator (`lib/prompts/index.ts`)

```ts
export async function generateAllDrafts(ctx: GenerationContext) {
  const [reddit, hn, ih, twitter, email] = await Promise.all([
    generateReddit(ctx),
    generateHackerNews(ctx),
    generateIndieHackers(ctx),
    generateTwitter(ctx),
    generateEmail(ctx),
  ]);

  return { reddit, hackernews: hn, indiehackers: ih, twitter, email };
}
```

Run all 5 in parallel. Total wall-clock should be ~10–15 seconds with `claude-sonnet-4-6`.

---

## Versioning the prompts

When you change a prompt:
1. Bump a version constant at the top of the file (e.g. `export const VERSION = '1.3'`)
2. Store the version on the `drafts.content` JSON
3. This lets you measure quality changes over time

When users complain a draft is bad:
1. Get the draft ID
2. Check which prompt version generated it
3. Diff against the current version to see if you've already fixed it

---

## How to evolve the prompts

1. **Read every Reddit/HN comment from real launches.** What got removed? Why?
2. **Save 10 great Reddit posts and 10 great Show HN posts as ground truth.** When changing a prompt, re-generate against the same input and verify quality didn't drop.
3. **Watch for drift.** As Claude models update, the same prompt produces different output. Re-test monthly.
4. **Don't let users override prompts.** They will water them down. The opinionated prompts ARE the product.
