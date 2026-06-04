# Investom — Indian Stock Analysis Platform: Architecture & Tech Stack

> Complete code layer breakdown for all 4 modules: Discover, Analyze, Evaluate, and Portfolio.

---

## Platform Overview

The platform is divided into **4 core modules**, each powered by AI:

| Module | Purpose |
|--------|---------|
| **Discover** | Find stocks, screen by criteria, watchlists, market heatmaps |
| **Analyze** | Deep fundamental + technical analysis, annual report RAG Q&A, peer comparison |
| **Evaluate** | Valuation models (DCF), risk scoring, sentiment analysis, investment thesis |
| **Portfolio** | P&L tracking (XIRR), attribution analysis, AI rebalancing, trade journal |

---

## Architecture Diagram

```
User Browser / Mobile App
        |
        ▼
  Frontend (Next.js 14)
        |
        ▼
  API Gateway (Node.js / Fastify)
        |
   ┌────┴────────────────────┐
   ▼                         ▼
AI Microservices         Market Data Layer
(Python / FastAPI)       (NSE/BSE APIs)
   |                         |
   ▼                         ▼
LLM APIs               PostgreSQL + Redis
(Claude / OpenAI)      + Vector DB (pgvector)
```

---

## Layer 1: Frontend

**Language:** TypeScript (strict mode)  
**Framework:** Next.js 14 (App Router)

### Libraries & Tools

| Tool | Purpose |
|------|---------|
| **Next.js 14** (App Router) | Core framework — SSR/SSG for SEO on stock pages |
| **TailwindCSS + shadcn/ui** | UI component system, consistent design language |
| **TradingView Lightweight Charts** | Candlestick / OHLCV charting for Analyze module |
| **Recharts / D3.js** | Portfolio pie charts, sector heatmaps, attribution graphs |
| **TanStack Query** | Server state management, caching, background price refetching |
| **Zustand** | Client-side state (watchlists, preferences, screener filters) |
| **React Hook Form + Zod** | Trade journal forms, screener inputs, schema validation |

### Key Screens

#### Module 1 — Discover
- AI Stock Screener UI with filter builder
- Market Heatmap (sector-wise color grid)
- Natural Language stock search bar
- Watchlist manager with AI entry alerts
- Stock Overview card (metrics snapshot)

#### Module 2 — Analyze
- Stock detail page with fundamental tabs
- Interactive OHLCV chart with indicators
- Annual Report RAG chat interface
- Earnings call transcript viewer with signal flags
- Peer comparison table

#### Module 3 — Evaluate
- Valuation dashboard (DCF + relative models)
- Risk score meter (5 dimensions)
- Sentiment gauge (−100 to +100)
- Bull / Bear thesis builder card
- Conviction score display

#### Module 4 — Portfolio
- Holdings table with unrealized P&L
- XIRR calculator and benchmark comparison
- Sector allocation donut chart
- AI rebalancing suggestion panel
- Trade journal with AI coaching notes

---

## Layer 2: Backend

Split into two services by concern:

### 2a. Node.js / Fastify — API Gateway & Business Logic

**Language:** TypeScript  
**Handles:** Auth, portfolio CRUD, watchlist management, market data aggregation, caching, job scheduling

| Library | Purpose |
|---------|---------|
| **Fastify** | High-performance REST API server |
| **Prisma** | Type-safe ORM for PostgreSQL |
| **Redis + BullMQ** | Cache live prices; queue heavy AI jobs (DCF, RAG indexing) |
| **Supabase Auth / Jose** | JWT-based authentication and session management |
| **Zod** | Runtime input validation on all API endpoints |

### 2b. Python / FastAPI — AI Microservices

**Language:** Python 3.11+  
**Handles:** LLM calls, RAG pipeline, sentiment analysis, technical indicators, valuation models

| Library | Purpose |
|---------|---------|
| **FastAPI** | AI service REST endpoints |
| **Anthropic SDK** | Claude API calls for all system prompts across 4 modules |
| **LangChain / LlamaIndex** | RAG orchestration for Annual Report Q&A |
| **TA-Lib / pandas-ta** | RSI, MACD, Bollinger Bands, moving averages, pattern detection |
| **numpy-financial** | XIRR computation for portfolio P&L tracking |
| **sentence-transformers** | Embedding annual report chunks for vector search |
| **pandas / numpy** | Financial data processing and transformations |

---

## Layer 3: Database

### Primary Database: PostgreSQL (via Supabase)

Supabase provides hosted PostgreSQL + pgvector + Auth + Realtime subscriptions.

#### Schema Overview

```sql
-- User Management
users
user_profiles
user_risk_profiles (Conservative / Moderate / Aggressive)

-- Stock Master Data
stocks                   -- 5000+ NSE/BSE tickers, sector, market cap category
stock_prices_daily       -- OHLCV + computed technical indicators
stock_fundamentals       -- Quarterly P&L, balance sheet, cash flow (Ind AS)
corporate_actions        -- Splits, dividends, buybacks, rights issues

-- User Portfolio & Watchlists
watchlists
watchlist_items
portfolios
holdings
transactions             -- Every buy/sell with timestamp for XIRR

-- AI Output Cache
analysis_cache           -- (stock_id, module, json_result, computed_at)
thesis_store             -- Bull/bear thesis per stock per user
conviction_scores
risk_scores
sentiment_snapshots

-- RAG Pipeline
document_chunks          -- (stock_id, doc_type, content, embedding VECTOR(1536))
```

### Vector Database: pgvector (bundled with Supabase)

Used for Annual Report RAG Q&A — stores chunked embeddings of:
- Annual reports (PDF → text → chunks → embeddings)
- Investor presentations
- Earnings call transcripts

> Migrate to **Pinecone** if collection exceeds ~5M vectors at scale.

### Cache: Redis (Upstash)

| Data | TTL |
|------|-----|
| Live stock prices | 15 seconds |
| Screener results | 5 minutes |
| AI analysis outputs | 24 hours |
| Fundamental data | 24 hours |

---

## Layer 4: AI / LLM

### Models

| Model | Provider | Use Case |
|-------|----------|---------|
| **Claude 3.5 Sonnet** | Anthropic | Primary LLM — thesis generation, fundamental summaries, earnings analysis, conviction scoring, watchlist intelligence |
| **GPT-4o** | OpenAI | Fallback / secondary LLM when Claude is unavailable |
| **Whisper** | OpenAI | Transcribing earnings call audio files to text |
| **text-embedding-3-small** | OpenAI | Embedding annual report chunks for pgvector similarity search |

### AI Features per Module

| Module | AI Feature | Model |
|--------|-----------|-------|
| Discover | Natural language stock search, screener filter parsing, stock overview card | Claude |
| Discover | Market heatmap narrative summary | Claude |
| Analyze | Fundamental health score + narrative | Claude |
| Analyze | Technical pattern recognition summary | Claude |
| Analyze | Annual report Q&A (RAG) | Claude + pgvector |
| Analyze | Earnings call transcript analysis | Whisper → Claude |
| Analyze | Peer comparison narrative | Claude |
| Evaluate | DCF + relative valuation with assumptions | Claude |
| Evaluate | Risk scoring (5 dimensions) | Claude |
| Evaluate | News & social sentiment classification | Claude |
| Evaluate | Bull/Bear thesis builder | Claude |
| Evaluate | Conviction scoring (weighted by investor style) | Claude |
| Portfolio | P&L insights and XIRR narrative | Claude |
| Portfolio | Rebalancing recommendations | Claude |
| Portfolio | Performance attribution narrative | Claude |
| Portfolio | Trade journal AI coaching | Claude |
| Portfolio | Portfolio health check | Claude |

### RAG Pipeline (Annual Report Q&A)

```
PDF Annual Report
      |
      ▼
Text Extraction (PyMuPDF / pdfplumber)
      |
      ▼
Text Chunking (LlamaIndex — 512 token chunks, 50 token overlap)
      |
      ▼
Embedding (text-embedding-3-small)
      |
      ▼
Store in pgvector (Supabase)
      |
      ▼
User query → embed query → similarity search → top-k chunks
      |
      ▼
Claude: Answer from retrieved context only (no hallucination)
```

---

## Layer 5: External Data Integrations

### Market Data — Real-Time & Historical

| API | Data | Priority |
|-----|------|---------|
| **Upstox API** | Real-time quotes, Level 2 order book, OHLCV | Primary real-time |
| **Zerodha Kite Connect** | Tick data, historical OHLCV, instrument list | Alternative real-time |
| **NSE India public endpoints** | Index data, F&O OI data, corporate actions, bulk/block deals | Free, authoritative |
| **Screener.in** | 10-year fundamentals: P&L, balance sheet, ratios, cash flow | Best for fundamentals |
| **Tickertape API** | Composite scores, ETF data, peer sets | Good UX-ready data |
| **Alpha Vantage** | Historical OHLCV backup | Fallback only |

### News & Sentiment

| Source | Data | Method |
|--------|------|--------|
| **NewsAPI** | Financial news aggregation | REST API |
| **Economic Times / Mint / Moneycontrol** | Indian financial news | RSS feeds |
| **Business Standard** | Analyst reports, market news | RSS + scraping |
| **Twitter/X API v2** | Social sentiment on `$TICKER` and company mentions | REST API |
| **Reddit API** | r/IndiaInvestments, r/DalalStreet community sentiment | PRAW (Python) |

### Regulatory & Filing Data

| Source | Data |
|--------|------|
| **BSE Corporate Filing API** | Annual reports (PDF), quarterly results, investor presentations |
| **NSE Corporate Filings** | Earnings call transcripts, SEBI disclosures, shareholding patterns |
| **SEBI EDGAR-equivalent** | Insider trading, bulk deals, AGM results |

---

## Deployment Stack

| Layer | Service |
|-------|---------|
| **Frontend** | Vercel (Next.js, Edge Functions for API routes) |
| **Backend Node.js** | Railway / Render / AWS ECS |
| **Backend Python (AI)** | Modal.com (serverless GPU) or AWS Lambda (CPU-only) |
| **Database** | Supabase (PostgreSQL + pgvector + Auth + Realtime) |
| **Cache** | Upstash Redis (serverless Redis) |
| **Job Queues** | BullMQ on Railway |
| **Vector DB** | pgvector (Supabase) → Pinecone at scale |
| **CDN** | Cloudflare (cache static assets, stock logos) |
| **Monitoring** | Sentry (errors), Posthog (product analytics) |

---

## Build Roadmap

| Week | Focus Area | Deliverable |
|------|-----------|-------------|
| **1–2** | Data Foundation | Stocks master DB (5000+ tickers), daily OHLCV pipeline, fundamentals ingestion from Screener.in |
| **2–3** | Discover Module | AI screener, NL search, sector heatmap, stock overview card, watchlist |
| **3–5** | Analyze Module | Fundamental AI summaries, technical chart + indicators, annual report RAG pipeline |
| **5–6** | Evaluate Module | DCF valuation engine, risk scoring, sentiment pipeline, thesis builder |
| **6–7** | Portfolio Module | XIRR tracker, sector allocation, rebalancing AI, trade journal with AI coaching |
| **7–8** | AI Chat Layer | Global assistant with access to stock data + user portfolio context |
| **8** | Compliance & Launch | SEBI disclaimers on all AI outputs, beta with 50–100 users from Valuepickr/TradingQnA |

---

## Security Considerations (OWASP-aligned)

- **XSS Prevention:** Sanitize all LLM-generated content before rendering — never dangerously set innerHTML with AI output
- **Rate Limiting:** Apply per-user rate limits on all AI endpoints to prevent prompt injection and cost abuse
- **Secret Management:** All API keys (Anthropic, Upstox, Zerodha) stored as server-side env vars only — never exposed to the client
- **Authentication:** JWT sessions with short expiry + refresh tokens; all portfolio mutations require valid auth
- **CSRF Protection:** CSRF tokens on all state-mutating API endpoints
- **Input Validation:** Zod schemas on every API route input; reject malformed requests before reaching business logic
- **SEBI Disclaimer:** Every AI-generated analysis output must carry: *"For informational purposes only. Investom is not a SEBI-registered investment advisor. This is not financial advice."*

---

## Summary: Technology Choices at a Glance

| Layer | Primary Choice | Why |
|-------|---------------|-----|
| Frontend | Next.js 14 + TypeScript | SSR for SEO, App Router for layouts, type safety |
| UI | TailwindCSS + shadcn/ui | Fast to build, consistent, accessible |
| Charts | TradingView Lightweight Charts | Industry standard for financial charts |
| API Gateway | Node.js + Fastify | Fast, TypeScript-native, low overhead |
| AI Services | Python + FastAPI | Best ecosystem for ML/AI libraries (TA-Lib, LangChain, pandas) |
| LLM | Claude 3.5 Sonnet | Best for long-context financial document analysis |
| RAG | LlamaIndex + pgvector | Simple setup, no extra infra needed |
| Database | PostgreSQL (Supabase) | Relational data + vector search + auth in one service |
| Cache | Redis (Upstash) | Serverless, low latency for live price caching |
| Market Data | Upstox + Screener.in | Real-time + fundamentals best covered by this pair |
| Deployment | Vercel + Railway + Supabase | Fast setup, scalable, developer-friendly |
