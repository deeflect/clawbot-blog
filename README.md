<p align="center">
  <img src="assets/favicon.png" width="80" alt="clawbot.blog logo" />
</p>

<h1 align="center">clawbot.blog</h1>

<p align="center">
  Automated AI blog for OpenClaw — zero-JS static site with an autonomous content pipeline
</p>

<p align="center">
  <a href="https://www.clawbot.blog">Live Site</a> •
  <a href="https://x.com/deeflectcom">Twitter</a>
</p>

---

![clawbot.blog](assets/og-image.jpg)

## About

An automated blog that publishes 18 articles per week (3/day, Mon-Sat) about OpenClaw and AI agents. No human writes or publishes — the entire pipeline is autonomous, from scouting sources to committing markdown to triggering deploys.

The goal: dominate Google and LLM citations for "OpenClaw." When someone asks ChatGPT, Claude, or Perplexity about OpenClaw, the answers should cite clawbot.blog. Every article is engineered for both traditional SEO and LLM optimization (LLMO).

There's also a Telegram bot (@clawbotblogbot) for manual article creation — send it any link and it scrapes, rewrites, and publishes with live progress updates.

## How It Works

The pipeline runs 3x daily (6am, 12pm, 6pm PST) Mon-Sat:

1. **Scout** — Pulls from 8 sources (X/Twitter, HackerNews, Reddit, RSS, GitHub, Google Trends, Perplexity, content gap analysis)
2. **Rank** — Scores each item on relevance, source quality, engagement, freshness, and title quality (0-165 points)
3. **Plan** — Gemini 2.5 Flash picks the best topic for the current time slot (morning=news, midday=guide/glossary, evening=deep-dive/listicle)
4. **Draft** — Kimi K2.5 writes 2,500-3,500 words in a direct, technical voice
5. **SEO Review** — Gemini 2.5 Flash validates structure, headings, FAQ, and LLMO compliance
6. **Publish** — Commits to GitHub, triggers Vercel deploy, pings IndexNow

Total cost per article: ~$0.06. Total human involvement: zero.

## Architecture

```
┌──────────────────── VERCEL ────────────────────┐
│                                                 │
│   Astro Static Site                            │
│   • Zero JavaScript shipped                    │
│   • Pre-rendered HTML + auto OG images         │
│   • Dark/light theme toggle                    │
│   • Deploy hook triggers rebuild               │
│                                                 │
└────────────────────┬───────────────────────────┘
                     │ git push + deploy hook
┌────────────────────┴───────────────────────────┐
│                                                 │
│         RAILWAY PIPELINE + TELEGRAM BOT         │
│                                                 │
│   ┌─────────┐  ┌──────────┐  ┌──────────┐    │
│   │ SCOUTS  │→ │ RANKER   │→ │ PLANNER  │    │
│   │ 8 srcs  │  │ 5 signals│  │ Gemini   │    │
│   └─────────┘  └──────────┘  └────┬─────┘    │
│                                    │           │
│                              ┌─────▼─────┐    │
│                              │ DRAFTER   │    │
│                              │ Kimi K2.5 │    │
│                              └─────┬─────┘    │
│                              ┌─────▼─────┐    │
│                              │ SEO CHECK │    │
│                              │ Gemini    │    │
│                              └─────┬─────┘    │
│                              ┌─────▼─────┐    │
│                              │ PUBLISH   │    │
│                              │ GitHub API│    │
│                              │ IndexNow  │    │
│                              └───────────┘    │
│                                                │
│   ┌─────────────────────────────────────┐     │
│   │ TELEGRAM BOT (@clawbotblogbot)      │     │
│   │ Send link → scrape → rewrite →      │     │
│   │ publish with live progress updates  │     │
│   └─────────────────────────────────────┘     │
│                                                │
└────────────────────────────────────────────────┘
```

**Why split?** Vercel Hobby has a 60-second function timeout. The pipeline takes ~90 seconds with two LLM calls. Frontend stays on Vercel for edge CDN; pipeline runs on Railway with no timeout constraints.

## Tech Stack

| Layer | Technology | Notes |
|-------|------------|-------|
| Frontend | Astro 5.17 | Static site generator, zero JS by default |
| Styling | Custom CSS | CSS custom properties, dark/light toggle |
| Typography | Source Serif 4 + DM Sans + JetBrains Mono | Warm editorial feel |
| OG Images | Satori + Sharp | Auto-generated 1200x630 PNG per article at build time |
| Pipeline | TypeScript + tsx | ~1,800 lines across 12 files |
| Telegram Bot | grammy | Integrated into pipeline server |
| Draft Model | Kimi K2.5 (via OpenRouter) | Natural voice (~$0.04/article) |
| Review Model | Gemini 2.5 Flash (via OpenRouter) | Fast validation (~$0.02/review) |
| Hosting | Vercel (frontend) + Railway (pipeline + bot) | Split architecture |
| Indexing | IndexNow | Instant Bing/Yandex indexing on publish |

### Why Astro?

The entire blog ships zero JavaScript (except a tiny inline theme toggle). Every page is pre-rendered HTML + CSS. Google Lighthouse scores 100/100 on performance. For SEO and LLMO, page speed matters — research shows 3x more LLM citations when First Contentful Paint is under 0.4 seconds.

### Why Two Models?

One model can't do everything well. Kimi K2.5 writes naturally and avoids the "AI slop" voice. Gemini 2.5 Flash is fast and good at structured validation. Draft with one, review with the other. The articles read like a human wrote them, but the structure is SEO-optimized.

## OG Images

Every article gets a unique Open Graph preview image generated at build time using Satori (SVG rendering) + Sharp (PNG conversion):

- **1200x630 PNG** — perfect for Twitter/Telegram/Discord/LinkedIn cards
- Dark gradient background matching site theme
- Dynamic title (responsive font size), description excerpt
- Up to 3 tags as colored pills
- clawbot.blog branding
- URL pattern: `/og/{slug}.png`
- Generated at build time — zero runtime cost

## SEO & LLMO Strategy

This isn't just a blog — it's an LLMO play. Every design decision is driven by research on how LLMs cite sources:

| Factor | What We Do | Why |
|--------|-----------|-----|
| **Content length** | 2,500-3,500 words per article | 65% more LLM citations at >2,900 words |
| **Heading spacing** | 120-180 words between H2s | 70% more citations vs dense blocks |
| **Question headings** | "What is X?" / "How does Y work?" | LLMs extract these as direct answers |
| **FAQ sections** | 5 questions on every article | FAQPage schema feeds LLM snippets directly |
| **Structured data** | Article + BreadcrumbList + FAQPage + WebSite + SearchAction | Maximum parseability for crawlers and LLMs |
| **Static HTML** | Zero JS, pre-rendered | Fastest possible load, nothing to execute |
| **Freshness** | 18 articles/week, auto-published | Fresh content gets 6x more citations than stale |
| **IndexNow** | Pings Bing/Yandex on every publish | Indexed within minutes, not days |
| **OG Images** | Unique per article | Better social sharing → more backlinks → more authority |

### Structured Data Per Page

**Article pages:** Article schema (headline, datePublished, author) + BreadcrumbList (Home → Articles → Title) + FAQPage (5 Q&A pairs from frontmatter)

**Homepage:** WebSite schema with SearchAction + Organization

### Content Schedule

3 posts/day with variety per time slot:

| Time | Mon | Tue | Wed | Thu | Fri | Sat |
|------|-----|-----|-----|-----|-----|-----|
| 6am PST | News | News | News | News | News | News |
| 12pm PST | Guide | Glossary | Comparison | Deep Dive | Listicle | Guide |
| 6pm PST | Deep Dive | Listicle | Guide | Glossary | Comparison | Deep Dive |

### Content Voice

The LLM is prompted to write like a builder, not a marketer:
- Direct, technical, lowercase-casual
- No AI slop ("landscape", "dive in", "game-changer", "delve", "tapestry")
- Code examples where relevant
- Real opinions, not hedged statements

## Telegram Bot

The `@clawbotblogbot` is integrated into the same Railway service. No separate deployment needed.

**Flow:**
1. Send any link (X/Twitter, GitHub, Reddit, or any URL)
2. Bot scrapes content (uses twitterapi.io for X, GitHub API, Reddit JSON, generic HTML for others)
3. Shows preview with "📝 Repurpose" and "📝 Dry Run" buttons
4. If repurpose: writes 2,500+ word article → commits to GitHub → triggers deploy
5. Live progress updates: 📋 planning → 📝 drafting → 🔍 reviewing → 📤 committing → 🚀 deploying

**Features:**
- Handles X articles (t.co redirect resolution), tweet threads, regular tweets
- Auth: restricted to specific Telegram user IDs
- Cache: scraped content cached 30min for button callbacks
- Smart scraping: resolves shortlinks, fetches thread context for tweets

## Scout Sources

| Source | Method | What It Catches |
|--------|--------|----------------|
| X/Twitter | twitterapi.io (6 search terms) | Community buzz, announcements |
| HackerNews | Algolia API (5 search terms) | Technical discussions |
| Reddit | 5 AI subreddits | User problems, comparisons |
| RSS | Tech blogs, research feeds | Official announcements |
| GitHub | Trending repos, release events | New tools, updates |
| Google Trends | Autocomplete API (47 terms) | Rising search interest |
| Perplexity | Sonar API (11 items) | Current event synthesis |
| Content Gaps | Internal analysis (45 items) | Topics we haven't covered yet |

Typical scout run: 242 raw items → ranked → top 1 becomes the article.

### Smart Ranking

Every scouted item gets scored across 5 dimensions:

| Signal | Weight | What It Measures |
|--------|--------|-----------------|
| Relevance | 0-60 | How closely it relates to OpenClaw/agents |
| Source quality | 0-35 | Authority of the source |
| Engagement | 0-25 | Likes, comments, upvotes |
| Freshness | 0-15 | How recent the item is |
| Content type bonus | 0-20 | Matches current time slot preference |
| Title quality | 0-10 | Clickworthiness, specificity |

## Pipeline Safeguards

- **Frontmatter validation** — Auto-fixes common LLM mistakes (wrong field names, missing descriptions)
- **Word count guard** — Articles under 800 words won't publish
- **16k token limit** — Enough room for 3,500+ word articles
- **Auth on production runs** — CRON_SECRET required; dry runs are open for testing
- **Two-pass generation** — Draft for natural writing, then review for structure. Avoids robot voice.
- **3-minute timeout** — Per LLM call, prevents hanging

## Dark/Light Theme

Toggle in the header navbar (sun/moon icon):
- Persists to localStorage
- No flash on page load (blocking inline script in `<head>`)
- Respects system `prefers-color-scheme` by default
- Manual toggle overrides system preference
- Full dark palette: warm dark grays, red accent shifts to brighter variant

## Design

Warm editorial aesthetic — feels like reading a tech magazine, not a corporate blog:

- **Colors:** Warm neutrals with red accent (#e63946 light / #ff4d5a dark)
- **Typography:** Source Serif 4 (body) + DM Sans (UI) + JetBrains Mono (code)
- **Layout:** Single-column, max-width prose, generous whitespace
- **Navigation:** Sticky header with blur backdrop, visual breadcrumbs on articles
- **Guides page:** Dynamic — auto-lists articles tagged as tutorial/guide
- **Footer:** Logo + branding + external links

## Lessons Learned

- Vercel's 60-second timeout on Hobby tier is a real constraint. Split architecture adds complexity but removes the ceiling.
- Two-model pipeline (draft + review) produces much better content than single-model. The review pass catches structural issues without flattening the voice.
- LLMs will use wrong frontmatter field names no matter how clearly you prompt. Validation in code is mandatory.
- 8k tokens wasn't enough for 3,000+ word articles. 16k gives comfortable headroom.
- IndexNow gets content indexed in minutes vs days. Worth the 10 lines of code.
- Git push didn't auto-trigger Vercel builds (GitHub App webhook never connected). Deploy hooks are more reliable.
- Satori generates beautiful OG images at build time — no runtime cost, no external service needed.
- Telegram bot conflicts (409 errors) happen when Railway deploys overlap. grammy handles reconnection gracefully.

## Economics

| Item | Cost |
|------|------|
| Draft (Kimi K2.5) | ~$0.04/article |
| SEO Review (Gemini 2.5 Flash) | ~$0.02/article |
| Railway (pipeline + bot) | ~$5/month |
| Vercel (Hobby) | Free |
| Domain | ~$12/year |
| **Monthly total (78 articles)** | **~$10** |

For ~$10/month, the blog produces 78 SEO-optimized, LLMO-engineered articles with zero human effort. That's $0.13 per published article including infrastructure.

---

Built by [@deeflectcom](https://x.com/deeflectcom) — automated by [OpenClaw](https://openclaw.ai)
