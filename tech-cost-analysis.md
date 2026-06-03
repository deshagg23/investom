# Investom — Technology Evaluation, Cost Analysis & Best Practices

> Comprehensive review of every technology choice, cost projection at three scale milestones, and actionable optimisation recommendations.

---

## 1. Technology Evaluation

### 1.1 Frontend

| Technology | Current Choice | Verdict | Recommendation |
|------------|---------------|---------|----------------|
| **Framework** | Next.js 14 (App Router) | ✅ Best choice | Upgrade to **Next.js 15** — stable, faster builds, improved caching, React 19 compatibility. Keep App Router. |
| **Styling** | TailwindCSS v3 | ✅ Best choice | Lock to v3. TailwindCSS v4 is in beta — do not use in production yet. |
| **UI Components** | shadcn/ui | ✅ Best choice | Built on Radix UI, fully accessible (WCAG 2.1), zero runtime overhead, copy-paste model means no vendor lock-in. |
| **Financial Charts** | TradingView Lightweight Charts | ✅ Best choice | Free, MIT licence, 60fps rendering, purpose-built for financial data. **Do NOT use TradingView widget embeds** — those require a $X,000/month white-label licence for commercial apps. |
| **Portfolio Charts** | Recharts + D3 | ⚠️ Redundant pair | Pick one. **Recharts** for all standard charts (donut, bar, line). Add **D3** only if you need custom force-directed or heatmap charts. Carrying both doubles your bundle size. |
| **Server State** | TanStack Query v5 | ✅ Best choice | Best-in-class for stale-while-revalidate, polling, and cache invalidation. |
| **Client State** | Zustand | ✅ Best choice | Minimal, no boilerplate, works well with TanStack Query. No need for Redux. |
| **Forms & Validation** | React Hook Form + Zod | ✅ Best choice | Industry standard. Zod schema doubles as runtime validation and TypeScript type source. |
| **Real-time** | Socket.io + Supabase Realtime | ❌ Redundant — double cost | **Remove Socket.io entirely.** Supabase Realtime (built on Phoenix/Elixir) handles all real-time needs and is already included in your Supabase plan. Running both adds infrastructure overhead with zero benefit. |

**Key Frontend Security Rules:**
- Never expose API keys in Next.js client components — use server actions or route handlers
- Apply `Content-Security-Policy` headers via `next.config.ts`
- Use `next/headers` to set `Strict-Transport-Security`, `X-Frame-Options`, `X-Content-Type-Options`
- Sanitise all user inputs before passing to AI prompts (prompt injection prevention)

---

### 1.2 Backend & Infrastructure

| Technology | Current Choice | Verdict | Recommendation |
|------------|---------------|---------|----------------|
| **API Gateway** | Node.js + Fastify | ✅ Good choice | Fastify is ~2x faster than Express and has built-in schema validation (JSON Schema). Keep it. |
| **AI Microservices** | Python + FastAPI | ✅ Best choice | FastAPI is the standard for Python AI services — async, type-safe, auto-generates OpenAPI docs. |
| **Primary Database** | PostgreSQL via Supabase | ✅ Best choice | Supabase gives you Postgres + pgvector + Auth + Storage + Realtime + Edge Functions in one platform. Best value for startup. |
| **Caching** | Redis (self-managed) | ⚠️ Overengineered for early stage | Replace with **Upstash Redis** (serverless, pay-per-request). Zero server management, free up to 10k req/day, scales automatically. Self-managed Redis adds $10-30/month server cost + maintenance before you need it. |
| **Job Queues** | BullMQ (requires Redis) | ✅ Good choice | Keep BullMQ — it's the best Node.js queue library. Switch its backing Redis to Upstash. |
| **Vector Database** | Pinecone + pgvector | ❌ Redundant — drop Pinecone | **pgvector within Supabase handles everything you need** up to ~5M vectors (the entire NSE/BSE stock universe with annual reports is well under this). Pinecone costs $70+/month for the Starter plan. Zero benefit at this scale. |
| **WebSockets** | Standalone WebSocket server | ❌ Unnecessary | Covered by Supabase Realtime. Remove. |
| **Deployment — Frontend** | Vercel | ✅ Best choice | Best Next.js hosting. Automatic edge caching, preview deployments, analytics. |
| **Deployment — Node.js** | Railway | ✅ Good choice | Railway is simpler and cheaper than AWS at early stage. $5/month Starter covers most needs. |
| **Deployment — Python AI** | Railway / Render | ⚠️ Better option exists | Use **Modal.com** for Python AI services. Pay only for actual compute used (~$0.03/CPU-hour). A Python service that runs 30 minutes/day costs ~$1/month on Modal vs $7-25/month on Render 24/7. |
| **CDN** | Cloudflare | ✅ Best choice | Free tier covers all static asset delivery needs. Also provides DDoS protection, bot management, and WAF rules at no cost. |

**Key Backend Security Rules:**
- Rate-limit all public API endpoints (Fastify `fastify-rate-limit` plugin) — 60 req/min per IP for public, 300 req/min per authenticated user
- All AI prompt inputs must be sanitised to prevent prompt injection — strip HTML, limit length, reject known jailbreak patterns
- Store secrets in environment variables only — use **Doppler** or **Vercel environment variables** — never commit to git
- Use `helmet` middleware in Fastify for HTTP security headers
- Row Level Security (RLS) must be enabled on every Supabase table
- Audit log all financial data mutations (portfolio buy/sell entries)

---

### 1.3 AI / LLM Layer

| Technology | Current Choice | Verdict | Recommendation |
|------------|---------------|---------|----------------|
| **Primary LLM** | Claude 3.5 Sonnet (all tasks) | ⚠️ Too expensive for all tasks | Use a **two-tier LLM strategy** (see Section 2.3). Sonnet only for complex analysis. Haiku for simple Q&A and digests. This cuts AI costs by 70-80%. |
| **RAG Framework** | LangChain + LlamaIndex | ⚠️ LangChain is problematic | **Drop LangChain.** It has major API instability across versions, over-abstraction, and hidden complexity. Use **LlamaIndex only** for RAG pipeline, or write a lightweight custom retrieval pipeline (50 lines of Python). LlamaIndex is more focused and production-stable. |
| **Vector DB for RAG** | Pinecone / pgvector | ❌ Drop Pinecone | pgvector in Supabase is sufficient. Cosine similarity on 1M vectors responds in <100ms. |
| **Transcription** | OpenAI Whisper | ✅ Good choice | For cost optimisation, use **Whisper large-v3 via Replicate or Groq** (~$0.0001/minute) instead of OpenAI Whisper API ($0.006/minute). 60x cheaper for earnings call transcription. |
| **Embeddings** | OpenAI text-embedding-3-small | ✅ Best value | Cheapest high-quality embedding model at $0.02/MTok. Keep this. |
| **LLM Fallback** | GPT-4o | ✅ Good fallback | Keep as fallback. But consider **GPT-4o-mini** as the fallback for simple tasks (18x cheaper than GPT-4o). |
| **Prompt Caching** | Not mentioned | ❌ Missing — critical cost saver | **Enable Anthropic prompt caching.** System prompts repeated across requests are cached for 5 minutes at 90% cost reduction. This is the single biggest AI cost optimisation available. |

**Recommended AI Routing Strategy:**

```
User Query → Query Classifier
    ├── Simple (definition, price, basic stat) → Claude 3.5 Haiku
    ├── Medium (comparison, news summary, screener) → Claude 3.5 Haiku
    ├── Complex (DCF valuation, thesis, RAG Q&A) → Claude 3.5 Sonnet
    └── Digest generation (daily batch) → Claude 3.5 Haiku (batch mode)
```

---

### 1.4 Market Data APIs — Critical Issues

> This is the **highest-risk area** of the current tech stack. Several named sources either have no official API, are ToS-prohibited for commercial use, or have unreliable uptime. These must be addressed before any development begins.

| API | Current Plan | Issue | Recommendation |
|-----|-------------|-------|----------------|
| **Upstox API** | Real-time quotes | ⚠️ Developer program requires individual broker account — not designed for serving data to many end users | Use for **developer/internal testing only**. For user-facing real-time data, use a licensed data vendor (see below). |
| **Zerodha Kite Connect** | Historical OHLCV | ⚠️ Same restriction — each user must have a Zerodha account linked | Same as above. Use for portfolio auto-import in R5 (user-initiated OAuth) — not for serving general market data. |
| **NSE India (public endpoints)** | Index data, F&O OI | ❌ No official public API — NSE blocks scrapers, changes page structure without notice, rate-limits aggressively | Use **NSE's official licensed data products** (see below) or a licensed vendor. |
| **Screener.in** | 10-year fundamentals | ❌ No official API — scraping violates their ToS for commercial use | Subscribe to **Screener.in's paid data service** (~₹5,000–15,000/month for API-like access) OR use **EODHD** for fundamentals. |
| **Tickertape API** | Composite scores, peer sets | ❌ Not a public API — Tickertape is a B2C product, not a data provider | Remove from stack. Source this data independently. |
| **Alpha Vantage** | Backup historical data | ⚠️ Free tier: 25 calls/day — insufficient for production. Premium ($50/month) has 500 calls/day which is still limited. | Use as tertiary fallback only. |
| **Chittorgarh / IPO Watch** | GMP data | ⚠️ No official API — GMP data is unregulated, unofficial data | Keep as scrape/parse source but add clear disclaimer in UI: "GMP is unofficial market data and is not verified." |

**Recommended Market Data Stack (production-ready):**

| Data Category | Recommended Source | Cost | Notes |
|--------------|-------------------|------|-------|
| **Real-time quotes (NSE/BSE)** | **Dhan Developer API** or **Fyers API** | Free (developer program) | Both are SEBI-regulated brokers with free developer APIs. Dhan has better documentation. Apply for developer access. |
| **Historical OHLCV (all stocks)** | **EODHD.com** (End of Day Historical Data) | $19/month (All World plan) | Covers 5000+ Indian stocks. Reliable, documented API, proper SLA. 20+ years of history. |
| **Fundamentals (P&L, B/S, ratios)** | **EODHD Fundamentals** | Included in $19 plan | P&L, Balance Sheet, Cash Flow, 10+ year history |
| **Corporate filings / announcements** | **BSE India Developer Portal** | Free | Official XBRL filings, quarterly results, board meetings, corporate actions |
| **NSE official data** | **NSE Data Products** (licensed) | ₹15,000–50,000/year | For index constituent data, F&O OI, and official corporate actions |
| **IPO data (calendar, subscription)** | **NSE IPO data + BSE IPO page** | Free (official) | Parse official IPO pages; supplement with Chittorgarh for GMP with disclaimer |
| **Sector / index constituents** | **EODHD** | Included | Nifty 50, Nifty 500, sector indices |
| **Annual reports (PDFs)** | **BSE corporate filings** | Free | DRHP, RHP, Annual Reports all filed here officially |

---

### 1.5 Notifications & Communication

| Technology | Current Choice | Verdict | Recommendation |
|------------|---------------|---------|----------------|
| **Push Notifications** | Firebase Cloud Messaging (FCM) | ✅ Best choice | Free, reliable, handles both iOS and Android. No change needed. |
| **Email** | Resend / SendGrid | ⚠️ Pick one | **Use Resend** — modern API, TypeScript-native, generous free tier (3,000 emails/month), better developer experience than SendGrid. Drop SendGrid unless you already have it. |
| **In-app Realtime** | Supabase Realtime | ✅ Best choice | Already included in Supabase plan. |

---

### 1.6 Security Architecture — Missing Pieces

The current stack description does not address these critical security requirements:

| Area | Gap | Solution |
|------|-----|---------|
| **Secret management** | No mention of secret vault | Use **Doppler** (free tier) or Vercel/Railway environment variables. Never store API keys in code. |
| **Rate limiting** | Not defined | Fastify `@fastify/rate-limit`: 60 req/min (public), 300 req/min (auth), 10 req/min (AI endpoints) |
| **AI prompt injection** | No sanitisation layer | Strip HTML tags, limit user input to 500 chars for chat, reject inputs containing known jailbreak patterns |
| **SEBI compliance** | Partial | The platform presents data only (no advice) — this is legally safe. Community module must auto-block any content that looks like a buy/sell recommendation (AI moderation already planned in R4). |
| **Data licensing** | Not addressed | Displaying NSE/BSE financial data commercially requires data vendor agreements. Using EODHD's commercial licence covers this. |
| **Input validation** | Mentioned via Zod | Apply Zod validation on ALL API route inputs on the server side, not just frontend forms. |
| **SQL injection** | Implicit (Supabase handles) | Use Supabase parameterised queries only. Never interpolate user strings into raw SQL. |
| **Authentication** | Supabase Auth | Enable MFA option for users. Use Supabase Auth with JWT. Set short token expiry (15 min access token, 7 day refresh token). |
| **HTTPS** | Vercel handles | Enforce HSTS. All other services (Railway, Render) must have SSL certificates configured. |

---

## 2. Cost Analysis

> All costs are in **USD per month** unless noted. Exchange rate used: ₹84 = $1 (June 2026).
> Three scale milestones: **Early Stage** (500 DAU), **Growth** (5,000 DAU), **Scale** (25,000 DAU).

---

### 2.1 Infrastructure Costs

| Service | Plan | Early Stage (500 DAU) | Growth (5,000 DAU) | Scale (25,000 DAU) | Notes |
|---------|------|----------------------|-------------------|-------------------|-------|
| **Vercel** (Frontend) | Hobby → Pro → Enterprise | $0 | $20/month | $150/month | Pro needed when >3 team members or custom domains + analytics |
| **Supabase** (DB + Auth + Realtime) | Free → Pro → Team | $0 (Free, 500MB) | $25/month (Pro, 8GB) | $599/month (Team) | Pro is sufficient up to ~100k MAU and 8GB data |
| **Upstash Redis** (Cache) | Pay-per-request | ~$0–2/month | ~$15/month | ~$60/month | $0.20 per 100k requests; 10k/day free |
| **Railway** (Node.js API) | Starter → Pro | $5/month | $20/month | $60/month | Pro needed for autoscaling |
| **Modal.com** (Python AI) | Pay-per-second | ~$2/month | ~$20/month | ~$80/month | ~$0.03/CPU-hour; scales to zero |
| **Cloudflare** (CDN + WAF) | Free → Pro | $0 | $20/month | $20/month | Pro adds WAF rules and better analytics |
| **FCM** (Push) | Free | $0 | $0 | $0 | Firebase FCM is free at all scales |
| **Resend** (Email) | Free → Pro | $0 (3k/month free) | $20/month (50k/month) | $90/month (200k/month) | |
| **Doppler** (Secrets) | Free | $0 | $0 | $24/month | Team plan needed at scale |
| **GitHub** (Code) | Free | $0 | $0 | $4/month | |
| **Total Infrastructure** | | **~$7–10/month** | **~$120/month** | **~$1,087/month** | |

---

### 2.2 Market Data API Costs

| API | Purpose | Cost | Notes |
|-----|---------|------|-------|
| **EODHD.com** (All World plan) | Historical OHLCV + Fundamentals + Sectors | $19/month | Covers all 5,000+ Indian stocks. Best value data feed available. |
| **Dhan Developer API** | Real-time quotes (15-min delayed for free users) | $0 | Free developer access. Real-time for premium users via Dhan broker link. |
| **BSE Developer Portal** | Corporate filings, quarterly results, IPO data | $0 | Official, free. |
| **NSE Data Products** (basic) | Index constituents, F&O OI | ₹15,000/year (~$15/month) | Required for Nifty constituent data. Can defer to R2/R3. |
| **Screener.in** (data subscription) | Screener-grade fundamentals cross-check | ₹2,000–5,000/month (~$25–60/month) | Optional — EODHD covers most of this. |
| **Groq API** (Whisper transcription) | Earnings call audio transcription | ~$2–10/month | Whisper large-v3 at $0.0001/min audio. 100 earnings calls/month = $1. |
| **Total Market Data** | | **~$36–100/month** | EODHD alone covers 90% of needs. |

---

### 2.3 AI / LLM Costs

> This is the most variable and highest-risk cost category. The strategy below can save up to 80% vs using Claude Sonnet for everything.

#### Two-Tier LLM Strategy

**Tier 1 — Claude 3.5 Haiku** (fast, cheap): Used for all simple and medium-complexity tasks  
- Input: **$0.80/MTok** | Output: **$4/MTok** | Prompt cache write: $1/MTok | Cache read: $0.08/MTok

**Tier 2 — Claude 3.5 Sonnet** (powerful, expensive): Used only for complex analysis  
- Input: **$3/MTok** | Output: **$15/MTok** | Prompt cache write: $3.75/MTok | Cache read: $0.30/MTok

#### Per-Feature Token Budget & Cost

| Feature | Tier | Avg Input Tokens | Avg Output Tokens | Cost per Call | Notes |
|---------|------|-----------------|------------------|---------------|-------|
| Ask AI (simple Q&A) | Haiku | 600 | 250 | **$0.00053** | Cached system prompt saves 80% |
| Ask AI (portfolio Q&A) | Haiku | 1,500 | 400 | **$0.00280** | Portfolio data adds tokens |
| Stock screener (NL) | Haiku | 400 | 200 | **$0.00032** | Convert NL to SQL filter — simple task |
| Concept explainer | Haiku | 300 | 300 | **$0.00036** | Mostly template-driven |
| Fundamental AI summary | Sonnet | 3,000 | 1,000 | **$0.02400** | Cache output for 24h |
| Technical pattern summary | Haiku | 1,000 | 400 | **$0.00240** | Simpler analysis |
| Peer comparison narrative | Haiku | 2,000 | 500 | **$0.00360** | Mostly structured data |
| DCF Valuation | Sonnet | 2,500 | 1,200 | **$0.02550** | Complex reasoning — Sonnet required |
| Risk scoring | Haiku | 2,000 | 600 | **$0.00400** | Rule-based + AI summary |
| Scenario analysis (thesis) | Sonnet | 3,500 | 2,000 | **$0.04050** | Complex multi-scenario output |
| Research completeness score | Haiku | 500 | 300 | **$0.00044** | Simple scoring |
| Earnings call analyzer | Sonnet | 8,000 | 2,000 | **$0.05400** | Long transcript input |
| Annual report RAG Q&A | Sonnet | 3,000 | 800 | **$0.02100** | RAG context window |
| IPO data completeness | Haiku | 2,000 | 600 | **$0.00400** | DRHP data parsing |
| Daily morning digest | Haiku | 3,500 | 800 | **$0.00600** | Batch — run once per user per day |
| AI moderation (community) | Haiku | 300 | 100 | **$0.00028** | Simple classification task |

> **Prompt caching impact:** System prompts average 500–800 tokens. With caching enabled, repeated calls within 5 minutes pay only $0.08/MTok for the cached portion (vs $0.80/MTok). For high-frequency features (Ask AI, screener), this alone reduces AI cost by 60–70%.

#### Monthly AI Cost Projection

Assumptions per scale milestone:

| Usage Pattern | Early Stage (500 DAU) | Growth (5,000 DAU) | Scale (25,000 DAU) |
|--------------|----------------------|-------------------|-------------------|
| Ask AI queries/user/day | 5 | 8 | 10 |
| Deep analysis/user/week | 2 | 3 | 4 |
| Daily digest (50% opt-in) | 250/day | 2,500/day | 12,500/day |
| Earnings call analyses/month | 20 | 100 | 500 |
| IPO DRHP analyses/month | 5 | 20 | 80 |

| Cost Item | Early Stage | Growth | Scale |
|-----------|------------|--------|-------|
| Ask AI (Haiku, with caching) | ~$20/month | ~$160/month | ~$700/month |
| Deep analysis (Sonnet, 24h cache) | ~$30/month | ~$250/month | ~$1,100/month |
| Daily digest (Haiku batch) | ~$45/month | ~$450/month | ~$2,250/month |
| Earnings + IPO analysis (Sonnet) | ~$15/month | ~$75/month | ~$375/month |
| Embeddings (text-embedding-3-small) | ~$2/month | ~$15/month | ~$60/month |
| Transcription (Groq Whisper) | ~$2/month | ~$10/month | ~$50/month |
| **Total AI Cost** | **~$114/month** | **~$960/month** | **~$4,535/month** |

> **Without two-tier strategy** (all Sonnet, no caching): Early ~$420/month, Growth ~$3,500/month, Scale ~$17,000/month. The strategy reduces costs by **60–75%**.

---

### 2.4 Development Cost

#### Option A — Freelance Team (India-based)

| Role | Monthly Rate | Duration | Total |
|------|-------------|----------|-------|
| Senior Full-Stack (Next.js + Node.js) | ₹80,000–1,20,000/month | 7 months | ₹5.6–8.4 lakhs |
| AI/ML Engineer (Python + LLM + RAG) | ₹80,000–1,20,000/month | 5 months | ₹4–6 lakhs |
| Mid-level Frontend Developer | ₹50,000–80,000/month | 5 months | ₹2.5–4 lakhs |
| Part-time DevOps / Infrastructure | ₹30,000–50,000/month | 3 months | ₹0.9–1.5 lakhs |
| **Total Development** | | | **₹13–20 lakhs (~$15,500–23,800)** |

#### Option B — Accelerated Solo Build (1 developer + heavy AI tooling)

With Cursor IDE + Claude API + v0.dev, a single experienced developer can achieve ~50–60% of the effort reduction:

| Phase | Weeks | Key Deliverables |
|-------|-------|-----------------|
| Setup + R1 (Foundation + Hook) | 5 weeks | Auth, data pipeline, Discover, Ask AI, basic alerts |
| R2 (Analysis Power) | 5 weeks | Full analysis suite, RAG pipeline, valuation engine |
| R3 (Portfolio + Alerts) | 4 weeks | Portfolio tracker, drift analyser, advanced alerts, digest |
| R4 (Growth features) | 5 weeks | IPO, Learning Center, Community |
| R5 (Scale + Mobile) | 7 weeks | Mobile app, premium tier, broker linking |

**Total: 26 weeks / 6.5 months**  
**Cost: ₹7–10 lakhs for one senior developer** (40% cheaper than Option A)

#### Per-Module Development Effort

| Module | Frontend | Backend | AI/Prompts | Total Effort |
|--------|----------|---------|------------|-------------|
| Infrastructure + Auth | 1 week | 2 weeks | — | **3 weeks** |
| 1 — Discover | 2 weeks | 1 week | 0.5 weeks | **3.5 weeks** |
| 2 — Analyze | 2.5 weeks | 1.5 weeks | 1 week | **5 weeks** |
| 3 — Evaluate | 2 weeks | 1.5 weeks | 1 week | **4.5 weeks** |
| 4 — Portfolio | 2.5 weeks | 2 weeks | 0.5 weeks | **5 weeks** |
| 5 — Ask AI | 1.5 weeks | 1 week | 1 week | **3.5 weeks** |
| 6 — IPO Tracker | 1.5 weeks | 1 week | 0.5 weeks | **3 weeks** |
| 7 — Learning Center | 1.5 weeks | 0.5 weeks | 1 week | **3 weeks** |
| 8 — Smart Alerts | 1 week | 2 weeks | 0.5 weeks | **3.5 weeks** |
| 9 — Community | 2 weeks | 1.5 weeks | 0.5 weeks | **4 weeks** |
| QA + Deployment | 1 week | 1 week | — | **2 weeks** |
| **Total** | **19 weeks** | **15.5 weeks** | **6.5 weeks** | **~40 dev-weeks** |

---

### 2.5 Total Monthly Operating Cost Summary

| Category | Early Stage (500 DAU) | Growth (5,000 DAU) | Scale (25,000 DAU) |
|----------|----------------------|-------------------|-------------------|
| Infrastructure | $10/month | $120/month | $1,087/month |
| Market Data APIs | $36/month | $75/month | $115/month |
| AI / LLM | $114/month | $960/month | $4,535/month |
| Miscellaneous (domain, monitoring, etc.) | $20/month | $40/month | $100/month |
| **Total Monthly Burn** | **~$180/month** (~₹15,000) | **~$1,195/month** (~₹1,00,000) | **~$5,837/month** (~₹4,90,000) |

**Break-even analysis at ₹199/month premium subscription:**

| Scale | Monthly Revenue (20% conversion) | Monthly Cost | Monthly Profit |
|-------|----------------------------------|-------------|----------------|
| 500 DAU (100 paying) | ₹19,900 | ₹15,100 | **+₹4,800** |
| 5,000 DAU (1,000 paying) | ₹1,99,000 | ₹1,00,000 | **+₹99,000** |
| 25,000 DAU (5,000 paying) | ₹9,95,000 | ₹4,90,000 | **+₹5,05,000** |

---

## 3. Best Recommendations

### 3.1 Cost Optimisation (Immediate Impact)

**1. Enable Anthropic Prompt Caching — do this on day 1**  
System prompts for all 9 modules are 400–800 tokens each. Caching them across requests within a 5-minute window cuts input token costs by 90% on the cached portion. Implementation: add `cache_control: {"type": "ephemeral"}` to system message.  
**Estimated savings: 40–60% of total AI bill.**

**2. Use Claude 3.5 Haiku for 80% of all AI calls**  
Route complex reasoning tasks (DCF, thesis, RAG) to Sonnet. Route everything else (Q&A, screener, summaries, digests, moderation) to Haiku.  
**Estimated savings: 50–70% of total AI bill.**

**3. Cache AI outputs in Redis/Upstash**  
Fundamental analysis, valuation estimates, and sector summaries change at most once per day (after market close). Cache outputs with a 6-hour TTL.  
**Estimated savings: 70–80% reduction in repeat analysis calls.**

**4. Use EODHD for fundamentals instead of Screener.in**  
One $19/month subscription replaces a patchwork of scrapers and unofficial API calls. Licensed, reliable, with SLA.

**5. Replace Pinecone with pgvector in Supabase**  
Saves $70+/month immediately. pgvector with HNSW index handles millions of vectors efficiently. Only reconsider Pinecone if you exceed 10M vectors or need multi-tenancy at scale.

---

### 3.2 Scalability Recommendations

**6. Design AI outputs as cacheable resources from day 1**  
Every AI output should have a cache key: `{feature}:{stock_ticker}:{date}:{user_style}`. This makes horizontal scaling trivial.

**7. Separate read and write paths in the database**  
Use Supabase read replicas (available on Pro plan) for all stock data queries. Write path only for user data (portfolio, alerts, watchlist). This alone can handle 10x traffic without changing architecture.

**8. Pre-compute daily digests overnight**  
Run BullMQ job at 2 AM to generate all user digests for 8 AM delivery. This spreads the AI cost across off-peak hours (some providers offer cheaper batch rates) and prevents thundering herd at 8 AM.

**9. Implement API response compression**  
Enable gzip/Brotli compression in Fastify. Stock fundamental data responses (large JSON) compress to 10–15% of original size, reducing bandwidth and CDN egress costs significantly.

**10. Use React Server Components for static-leaning pages**  
Stock overview cards, sector pages, and IPO calendar content changes infrequently. Render via Next.js RSC with `revalidate: 3600` (1 hour). Vercel edge cache serves these without hitting the API server.

---

### 3.3 Security Recommendations

**11. Implement AI prompt injection defence**  
Before passing user input to any AI model, run a lightweight classifier (can be Haiku itself, ~$0.0001 per check) that flags and blocks inputs matching injection patterns. Log all blocked attempts.

**12. Add SEBI compliance watermarking to AI outputs**  
Every AI-generated analysis page must display a non-dismissible banner: *"This is data and analysis only — not investment advice. The platform is not a SEBI-registered investment adviser."* Make this a layout component that cannot be overridden.

**13. Enable Supabase Row Level Security on every table**  
Ensure users can only read their own portfolio, watchlist, and alert data. Stock market data tables are read-only for all authenticated users. Community ideas are publicly readable but privately writable.

**14. Audit log for portfolio mutations**  
Every add/edit/delete of a portfolio holding must write to an immutable audit log table. This provides a paper trail and protects against data integrity issues.

---

### 3.4 Recommended Final Tech Stack (Revised)

| Layer | Technology | Change from Original |
|-------|-----------|---------------------|
| **Frontend** | Next.js 15, TailwindCSS v3, shadcn/ui, TradingView Lightweight Charts, **Recharts only** (drop D3), TanStack Query, Zustand | Drop D3; remove Socket.io |
| **Backend** | Node.js + Fastify, Python + FastAPI | No change |
| **Database** | PostgreSQL via Supabase (with pgvector, RLS, read replicas) | Drop Pinecone |
| **Cache** | **Upstash Redis** (serverless) | Replace self-managed Redis |
| **Queues** | BullMQ (backed by Upstash Redis) | Switch Redis backing only |
| **Real-time** | **Supabase Realtime only** | Remove Socket.io |
| **Primary LLM** | Claude 3.5 Sonnet (complex) + **Claude 3.5 Haiku** (simple) | Add Haiku tier |
| **LLM Fallback** | **GPT-4o-mini** | Downgrade fallback from GPT-4o |
| **RAG** | **LlamaIndex only** | Remove LangChain |
| **Transcription** | **Groq Whisper large-v3** | Replace OpenAI Whisper (60x cheaper) |
| **Embeddings** | OpenAI text-embedding-3-small | No change |
| **Market Data (real-time)** | **Dhan Developer API** | Replace Upstox/Zerodha for serving data |
| **Market Data (historical + fundamentals)** | **EODHD.com** ($19/month) | Replace Screener.in, Alpha Vantage |
| **Corporate filings** | **BSE India Developer Portal** (free) | Replace ad-hoc scraping |
| **Email** | **Resend** | Drop SendGrid |
| **Push** | Firebase Cloud Messaging | No change |
| **Deployment (Frontend)** | Vercel | No change |
| **Deployment (Node.js)** | Railway | No change |
| **Deployment (Python AI)** | **Modal.com** | Replace Render/Railway for Python |
| **CDN + Security** | Cloudflare (free tier) | No change |
| **Secrets** | **Doppler** | New addition |
| **Monitoring** | **Sentry** (free tier: 5k errors/month) + **Axiom** (free tier: 1GB/day logs) | New additions |

---

## 4. Quick Decision Reference

| Decision | Recommended Choice | Reason |
|----------|-------------------|--------|
| Which LLM for Q&A? | Claude 3.5 Haiku + prompt caching | 10x cheaper than Sonnet, sufficient quality for Q&A |
| Which LLM for deep analysis? | Claude 3.5 Sonnet + prompt caching | Reasoning quality matters here |
| Self-managed Redis vs serverless? | Upstash Redis | Zero management, pay-per-use, same API |
| Pinecone vs pgvector? | pgvector in Supabase | Free, sufficient for this scale, one less service |
| LangChain vs LlamaIndex vs custom? | LlamaIndex or lightweight custom | LangChain is over-engineered for this use case |
| Screener.in vs EODHD? | EODHD.com | Official API, licensed, $19/month covers all needs |
| Socket.io vs Supabase Realtime? | Supabase Realtime only | Already in stack, no additional server needed |
| OpenAI Whisper vs Groq Whisper? | Groq (Whisper large-v3) | 60x cheaper for transcription workloads |
| When to move to Scale tier? | At 5,000+ DAU | Under that, Pro/Starter plans handle everything |
| When to add a dedicated DBA? | At 25,000+ DAU or 50GB+ DB | Supabase managed handles everything before that |
| Mobile: React Native vs Flutter? | React Native / Expo | Code sharing with Next.js web app via shared logic |

---

*Last updated: June 2026 — Prices reflect current market rates and should be re-verified quarterly as LLM pricing continues to decline.*
