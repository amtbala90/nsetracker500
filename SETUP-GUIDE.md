# NSE Live Tracker — Full Setup & Run Guide

> **Complete instructions to download, install, configure, and run the NSE Live Stock Tracker application.**
> This guide is designed so you can hand this project to z.ai (or any developer) and have it running in minutes.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Technology Stack](#2-technology-stack)
3. [Prerequisites](#3-prerequisites)
4. [Project Structure](#4-project-structure)
5. [Installation Steps](#5-installation-steps)
6. [Running the Application](#6-running-the-application)
7. [How the Application Works](#7-how-the-application-works)
8. [Data Sources & Fallback Chain](#8-data-sources--fallback-chain)
9. [Environment Variables](#9-environment-variables)
10. [API Endpoints](#10-api-endpoints)
11. [Key Features Explained](#11-key-features-explained)
12. [Configuration & Customization](#12-configuration--customization)
13. [Troubleshooting](#13-troubleshooting)
14. [Prompt for z.ai to Run This App](#14-prompt-for-zai-to-run-this-app)

---

## 1. Project Overview

**NSE Live Tracker** is a real-time stock market intelligence dashboard that tracks **500+ NSE (National Stock Exchange of India) stocks** with the following capabilities:

- **Live 30-second auto-refresh** cycle with countdown timer
- **Rolling interval percentage changes** at 30s, 1m, 2m, 3m, 4m, 5m timeframes (NOT cumulative day change)
- **Volume-based analysis**: volume spike detection, price-volume divergence, breakout confirmation, and trade signal generation (BUY / SELL / WATCH / WAIT)
- **Support & Resistance levels** with Entry / Stop-Loss / Target price suggestions
- **Data maturity indicators** — shows "N/A" until enough real snapshots are accumulated (~5 minutes for all timeframes)
- **Data source transparency** — clearly indicates whether data is from Yahoo Finance (Live), NSE India (Live), or Simulated
- **Beautiful dark-theme UI** with sparkline charts, animated badges, hero cards for top gainers/losers, and real-time price flash effects
- **4 dashboard tabs**: Top Gainers, Top Losers, Analysis (charts & distribution), Volume Analysis (signals, breakouts, divergences)

---

## 2. Technology Stack

| Component | Technology | Version |
|---|---|---|
| **Framework** | Next.js (App Router) | 16.x |
| **Language** | TypeScript | 5.x |
| **Runtime** | Bun | 1.x |
| **React** | React | 19.x |
| **Styling** | Tailwind CSS 4 | 4.x |
| **UI Components** | shadcn/ui (New York style) | Latest |
| **Charts** | Recharts | 2.15.x |
| **Icons** | Lucide React | 0.525.x |
| **ORM** | Prisma (SQLite) | 6.x |
| **AI SDK** | z-ai-web-dev-sdk | 0.0.17 |
| **Reverse Proxy** | Caddy | — |
| **Output Mode** | Standalone | — |

---

## 3. Prerequisites

Before you begin, ensure the following are installed on your system:

| Requirement | Minimum Version | Check Command |
|---|---|---|
| **Node.js** | 18.x+ | `node --version` |
| **Bun** | 1.x+ | `bun --version` |
| **Git** | Any | `git --version` |

> **Note:** Bun is the primary package manager and runtime for this project. All scripts in `package.json` are designed to run with Bun. If you don't have Bun, install it first:
> ```bash
> curl -fsSL https://bun.sh/install | bash
> ```

---

## 4. Project Structure

```
project-root/
├── .env                          # Environment variables (DATABASE_URL)
├── .gitignore                    # Git ignore rules
├── Caddyfile                     # Caddy reverse proxy config (port 81 → 3000)
├── bun.lock                      # Bun lockfile
├── components.json               # shadcn/ui configuration
├── eslint.config.mjs             # ESLint configuration
├── next.config.ts                # Next.js config (standalone output)
├── package.json                  # Dependencies & scripts
├── postcss.config.mjs            # PostCSS with Tailwind
├── tailwind.config.ts            # Tailwind CSS dark mode + shadcn vars
├── tsconfig.json                 # TypeScript config (@/ → ./src/*)
│
├── db/
│   └── custom.db                 # SQLite database file
│
├── prisma/
│   └── schema.prisma             # Prisma schema (User, Post models)
│
├── public/
│   ├── logo.svg                  # App logo
│   └── robots.txt                # SEO robots file
│
├── src/
│   ├── app/
│   │   ├── globals.css           # Global styles (Tailwind + shadcn CSS vars)
│   │   ├── layout.tsx            # Root layout (dark mode, Geist fonts, Toaster)
│   │   ├── page.tsx              # Main dashboard page (~1944 lines)
│   │   └── api/
│   │       ├── route.ts          # Simple health-check API
│   │       └── nse/
│   │           └── route.ts      # Core backend API (635 lines)
│   │
│   ├── components/
│   │   └── ui/                   # 42 shadcn/ui components
│   │       ├── accordion.tsx
│   │       ├── alert.tsx
│   │       ├── badge.tsx
│   │       ├── button.tsx
│   │       ├── card.tsx
│   │       ├── chart.tsx
│   │       ├── dialog.tsx
│   │       ├── separator.tsx
│   │       ├── table.tsx
│   │       ├── tabs.tsx
│   │       ├── tooltip.tsx
│   │       └── ... (30+ more)
│   │
│   ├── hooks/
│   │   ├── use-mobile.ts         # Mobile breakpoint hook
│   │   └── use-toast.ts          # Toast notification hook
│   │
│   └── lib/
│       ├── db.ts                 # Prisma client singleton
│       ├── nse-api.ts            # Stock data fetching (758 lines)
│       └── utils.ts              # cn() utility (clsx + tailwind-merge)
│
└── download/                     # Generated files output directory
    └── README.md
```

### Key Files Explained

| File | Purpose | Lines |
|---|---|---|
| `src/lib/nse-api.ts` | Yahoo Finance API integration (primary), NSE India API (fallback), simulated data generator (last resort). Contains 435+ NSE stock symbols. | ~758 |
| `src/app/api/nse/route.ts` | Backend API route — maintains in-memory price+volume snapshots, calculates multi-timeframe changes, runs volume analysis, determines data maturity. | ~635 |
| `src/app/page.tsx` | Frontend dashboard — all UI components, charts, tables, real-time updates, 4 tabs (Gainers/Losers/Analysis/Volume Analysis). | ~1944 |
| `src/app/layout.tsx` | Root layout — dark mode by default, Geist fonts, page metadata, Toaster component. | ~41 |

---

## 5. Installation Steps

### Step 1: Clone or Download the Project

If you have the project as a zip file, extract it to your desired directory. If using Git:

```bash
git clone <repository-url>
cd project-directory
```

### Step 2: Install Dependencies

```bash
bun install
```

This will install all 80+ dependencies listed in `package.json`, including:
- `next` (16.x), `react` (19.x), `react-dom` (19.x)
- All `@radix-ui/*` components for shadcn/ui
- `recharts` for charts
- `lucide-react` for icons
- `prisma` and `@prisma/client` for database
- `z-ai-web-dev-sdk` for AI capabilities
- `tailwind-merge`, `clsx`, `class-variance-authority` for styling utilities

### Step 3: Set Up Environment Variables

The `.env` file should contain:

```env
DATABASE_URL=file:/home/z/my-project/db/custom.db
```

> **Note:** Adjust the path if your project is in a different directory. The database is SQLite — no external database server needed.

### Step 4: Initialize the Database

```bash
bun run db:push
```

This creates the SQLite database schema based on `prisma/schema.prisma`. The schema includes `User` and `Post` models (not actively used by the stock tracker but required by the project scaffold).

### Step 5: Verify Installation

```bash
bun run build
```

If the build completes without critical errors, your installation is successful. (TypeScript errors are ignored via `ignoreBuildErrors: true` in `next.config.ts`.)

---

## 6. Running the Application

### Development Mode (Recommended)

```bash
bun run dev
```

This starts the Next.js development server on **port 3000**:
- URL: `http://localhost:3000`
- Hot-reload enabled
- Console logs visible in terminal
- API requests logged in real-time

### Production Mode

```bash
# Step 1: Build the application
bun run build

# Step 2: Start the production server
bun run start
```

The production server runs the standalone build on port 3000.

### With Caddy Reverse Proxy

If you want the app accessible on port 81 (as configured in `Caddyfile`):

```bash
# Start the app on port 3000
bun run dev

# In another terminal, start Caddy
caddy run
```

The app will be accessible at `http://localhost:81`.

### Accessing the Dashboard

Once running, open your browser and navigate to:

- **Direct**: `http://localhost:3000`
- **Via Caddy**: `http://localhost:81`

You should see the NSE Live Tracker dashboard with a dark theme, loading spinner, and then stock data populating within 30 seconds.

---

## 7. How the Application Works

### Data Flow Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                     Browser (Client Side)                        │
│                                                                  │
│  page.tsx ──fetch──→ /api/nse?type=gainers&refresh=true         │
│     │                                                            │
│     ├── 30-second auto-refresh (setInterval)                     │
│     ├── Flash effects on price changes                           │
│     ├── Sparkline charts (SVG polyline)                          │
│     ├── Recharts (AreaChart, BarChart, PieChart)                 │
│     └── 4 Tabs: Gainers | Losers | Analysis | Volume Analysis   │
└───────────────────────────┬──────────────────────────────────────┘
                            │
                            ▼
┌──────────────────────────────────────────────────────────────────┐
│                  Backend API (Server Side)                        │
│                                                                  │
│  /api/nse/route.ts (GET handler)                                 │
│     │                                                            │
│     ├── 1. Fetch from Yahoo Finance (435+ stocks, batched 100)   │
│     │      URL: query1.finance.yahoo.com/v7/finance/quote        │
│     │                                                            │
│     ├── 2. Fallback: NSE India API (top gainers/losers)          │
│     │      URL: www.nseindia.com/api/...                         │
│     │                                                            │
│     ├── 3. Last Resort: Simulated data generator                 │
│     │                                                            │
│     ├── Maintain in-memory Map<string, StockRecord>              │
│     │   └── Price+Volume snapshots every 30s (keep 20 min)      │
│     │                                                            │
│     ├── Calculate rolling % changes for 30s/1m/2m/3m/4m/5m      │
│     │   Formula: ((current - past) / past) × 100                 │
│     │   Returns null if insufficient history                     │
│     │                                                            │
│     ├── Determine data maturity (SEEDING vs MATURE)              │
│     │   MATURE when ≥5 min of real snapshots exist               │
│     │                                                            │
│     ├── Run volume analysis per stock:                           │
│     │   ├── Volume Ratio (current interval / avg interval)       │
│     │   ├── Volume Spike detection (>2x average)                 │
│     │   ├── Volume Trend (rising/falling/stable)                 │
│     │   ├── Price-Volume Divergence (bullish/bearish)            │
│     │   ├── Volume Breakout (price above resistance + vol spike) │
│     │   ├── Support (day low) / Resistance (day high)            │
│     │   ├── Trade Signal (BUY/SELL/WATCH/WAIT)                   │
│     │   └── Entry / Stop-Loss / Target prices                    │
│     │                                                            │
│     └── Return JSON response with data + meta + market status    │
└──────────────────────────────────────────────────────────────────┘
```

### Data Maturity Lifecycle

1. **First fetch** (Cycle 0): Each stock gets a single price snapshot. All timeframe changes return `null` → UI shows "N/A" with hourglass icon.

2. **1-2 minutes**: 30-second and 1-minute timeframes start showing real values. 2m+ still "N/A".

3. **5+ minutes**: All timeframes (30s through 5m) have enough history. Stock transitions from `SEEDING` → `MATURE`. The maturity badge in the header updates accordingly.

4. **Ongoing**: Snapshots older than 20 minutes are pruned to prevent memory bloat. The rolling window always contains the latest 20 minutes of data.

### Volume Analysis Logic

The volume analysis engine runs on every 30-second refresh cycle for each stock:

| Metric | Calculation | Significance |
|---|---|---|
| **Volume Ratio** | Current 30s volume delta / Average 30s volume delta | How active the current interval is vs. typical |
| **Volume Spike** | Volume Ratio ≥ 2.0 | Unusual activity — potential catalyst |
| **Volume Trend** | Recent 3 intervals vs. older 3 intervals | Rising (>1.2x), Falling (<0.8x), or Stable |
| **Price-Volume Divergence** | Price direction vs. volume direction | Bearish div: price up + volume falling; Bullish div: price down + volume rising |
| **Volume Breakout** | Price near day high/low + volume spike | Bullish breakout: price ≥ resistance with vol confirmation; Bearish: price ≤ support with vol |
| **Support** | Day's low price | Price floor for the trading session |
| **Resistance** | Day's high price | Price ceiling for the trading session |
| **Trade Signal** | Combined from breakout + divergence + volume | BUY (bullish breakout/divergence), SELL (bearish), WATCH (vol spike only), WAIT (no signal) |
| **Entry Price** | Current LTP at time of signal | Where to enter the trade |
| **Stop Loss** | Support - 10% of range (for BUY) / Resistance + 10% of range (for SELL) | Risk management level |
| **Target Price** | LTP + range (for BUY) / LTP - range (for SELL) | Profit target based on day's range |
| **Confidence** | High (vol spike + breakout), Medium (breakout/divergence), Low (spike only) | Signal reliability indicator |

---

## 8. Data Sources & Fallback Chain

The application uses a 3-tier data source priority system:

### Priority 1: Yahoo Finance API (Primary)

- **URL**: `https://query1.finance.yahoo.com/v7/finance/quote`
- **Coverage**: 435+ NSE stocks (using `.NS` suffix)
- **Authentication**: None required (free, open API)
- **Rate Limiting**: 200ms delay between batch requests
- **Batch Size**: 100 symbols per request
- **Data Quality**: Real-time market data during trading hours
- **Indicator**: Green "Yahoo Finance (Live)" badge in header

### Priority 2: NSE India API (Fallback)

- **URL**: `https://www.nseindia.com/api/...`
- **Coverage**: Top gainers/losers (50 stocks max)
- **Authentication**: Session cookies (auto-managed)
- **Session TTL**: 5 minutes
- **Data Quality**: Real-time but limited coverage
- **Indicator**: Blue "NSE India (Live)" badge in header

### Priority 3: Simulated Data (Last Resort)

- **Source**: `generateSimulatedGainers()` function
- **Coverage**: 85 hardcoded stocks with realistic base prices
- **Data Quality**: Artificial — price movements are randomly generated
- **Indicator**: Amber "Simulated Data" badge + warning banner

> **Important**: When simulated data is active, a prominent amber warning banner appears at the top of the dashboard stating: "Simulated Mode: Live data unavailable. Showing generated data with artificial price movements."

---

## 9. Environment Variables

| Variable | Default | Description |
|---|---|---|
| `DATABASE_URL` | `file:/home/z/my-project/db/custom.db` | SQLite database connection string |

No API keys are required. The Yahoo Finance API is free and does not require authentication. The NSE India API manages its own session cookies internally.

---

## 10. API Endpoints

### `GET /api/nse`

The primary data endpoint that powers the entire dashboard.

**Query Parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `type` | string | `gainers` | `gainers` or `losers` — determines sort order |
| `refresh` | boolean | `false` | `true` forces a fresh data fetch from API sources |

**Response Structure:**

```json
{
  "success": true,
  "data": [
    {
      "symbol": "TATASTEEL",
      "open": 142.50,
      "high": 146.80,
      "low": 141.20,
      "ltp": 145.30,
      "previousClose": 140.10,
      "dayChange": 5.20,
      "dayPChange": 3.71,
      "change30s": 0.40,
      "pChange30s": 0.28,
      "change1m": 0.80,
      "pChange1m": 0.55,
      "change2m": null,
      "pChange2m": null,
      "change3m": null,
      "pChange3m": null,
      "change4m": null,
      "pChange4m": null,
      "change5m": null,
      "pChange5m": null,
      "totalTradedVolume": 52340000,
      "totalTradedValue": 7587620000,
      "lastPrice": 145.30,
      "yearHigh": 184.60,
      "yearLow": 114.60,
      "priceHistory": [141.2, 142.0, 143.5, 144.8, 145.3],
      "volumeHistory": [2300000, 1800000, 3200000, 2500000],
      "lastUpdated": "2026-04-18T10:30:00.000Z",
      "dataSource": "yahoo_live",
      "dataMaturity": "seeding",
      "volumeRatio": 2.3,
      "avgVolume": 1500000,
      "volumeSpike": true,
      "volumeSpikeMultiplier": 2.3,
      "priceVolumeDivergence": "none",
      "volumeBreakout": "bullish_breakout",
      "support": 141.20,
      "resistance": 146.80,
      "tradeSignal": "BUY",
      "entryPrice": 145.30,
      "stopLoss": 140.64,
      "targetPrice": 150.90,
      "confidence": "high",
      "volumeTrend": "rising",
      "relativeVolume": 87
    }
  ],
  "meta": {
    "totalStocks": 435,
    "totalSymbolsInUniverse": 435,
    "lastUpdated": "2026-04-18T10:30:00.000Z",
    "isLive": true,
    "dataSource": "yahoo_live",
    "dataSourceLabel": "Yahoo Finance (Live)",
    "dataMaturity": "seeding",
    "matureStocks": 50,
    "seededStocks": 385,
    "coverage": "435/435 (100.0%)",
    "nextRefresh": "2026-04-18T10:30:30.000Z",
    "refreshInterval": 30,
    "timeframes": ["30s", "1m", "2m", "3m", "4m", "5m"],
    "volumeSummary": {
      "buySignals": 12,
      "sellSignals": 5,
      "watchSignals": 8,
      "volumeSpikes": 15,
      "bullishBreakouts": 7,
      "bearishBreakouts": 3
    }
  },
  "marketStatus": [
    {
      "market": "Capital Market",
      "marketStatus": "Open",
      "index": "NIFTY 50",
      "last": 23456.70,
      "variation": 123.45,
      "percentChange": 0.53,
      "tradeDate": "18-APR-2026",
      "marketStatusMessage": "Normal Market is Open"
    }
  ]
}
```

### `GET /api`

Simple health-check endpoint that returns `{ "message": "Hello, world!" }`.

---

## 11. Key Features Explained

### Feature 1: Rolling Interval Percentage Changes

Unlike typical stock dashboards that show the day's cumulative change, this tracker calculates **rolling interval changes** — the percentage change from the price X minutes ago to the current price.

**Formula**: `((currentPrice - priceAtXMinutesAgo) / priceAtXMinutesAgo) × 100`

**Timeframes**: 30 seconds, 1 minute, 2 minutes, 3 minutes, 4 minutes, 5 minutes

**Honest N/A Display**: When insufficient data exists (e.g., the app just started), the change values show as "N/A" with an hourglass icon rather than fake or zero values. This typically resolves within 5 minutes as snapshots accumulate.

### Feature 2: Volume Spike Detection

The system computes the volume traded in each 30-second interval (delta between cumulative volume snapshots) and compares it to the running average. A **Volume Spike** is detected when the current interval's volume exceeds 2x the average, signaling unusual market activity that may precede a significant price move.

### Feature 3: Volume Breakout Strategy

Inspired by the Tata Steel example from real trading:

1. **Consolidation Range**: The day's low acts as Support, the day's high as Resistance.
2. **Breakout Trigger**: Price moves above resistance (or below support) with a volume spike (ratio ≥ 1.5x).
3. **Confirmation**: Both price AND volume must align — a price breakout without volume confirmation is not signaled.
4. **Trade Execution**:
   - **Entry**: Current market price at time of breakout
   - **Stop Loss**: Below support minus a safety margin (10% of the day's range for breakouts)
   - **Target**: Entry price plus the full day's range (resistance - support)

### Feature 4: Price-Volume Divergence

- **Bearish Divergence**: Price is rising but volume is falling → upward momentum may be weakening
- **Bullish Divergence**: Price is falling but volume is rising → selling pressure may be exhausting, reversal possible

### Feature 5: Data Source Transparency

The dashboard prominently displays:
- A **DataSource badge** in the header (green for Yahoo Live, blue for NSE Live, amber for Simulated)
- A **Maturity badge** (MATURE or SEEDING)
- Warning banners when data is simulated or still collecting
- Per-stock maturity indicators (hourglass icon next to symbol when seeding)

### Feature 6: Auto-Refresh & Countdown

- The backend fetches fresh data every **28 seconds** (2 seconds ahead of the 30-second client refresh)
- The frontend polls the API every **30 seconds**
- A live countdown timer shows seconds until the next refresh
- Price changes trigger a brief **green flash effect** on the affected stock row
- A cycle counter tracks how many refresh cycles have occurred

---

## 12. Configuration & Customization

### Changing the Refresh Interval

In `src/app/api/nse/route.ts`:
```typescript
const FETCH_INTERVAL = 28 * 1000; // Backend fetch interval (milliseconds)
```

In `src/app/page.tsx`:
```typescript
intervalRef.current = setInterval(() => {
  fetchData();
}, 30000); // Frontend refresh interval (milliseconds)
```

### Adding More Stock Symbols

In `src/lib/nse-api.ts`, add symbols to the `ALL_NSE_SYMBOLS` array:
```typescript
export const ALL_NSE_SYMBOLS: string[] = [
  // ... existing symbols ...
  'YOUR_NEW_SYMBOL',  // Add here
];
```

The system automatically deduplicates and appends `.NS` suffix for Yahoo Finance API calls.

### Changing Volume Spike Threshold

In `src/app/api/nse/route.ts`:
```typescript
const volumeSpike = volumeRatio !== null && volumeRatio >= 2.0; // Change 2.0 to desired threshold
```

### Changing Data Maturity Threshold

In `src/app/api/nse/route.ts`:
```typescript
function determineMaturity(snapshots: PriceSnapshot[]): DataMaturity {
  // ...
  return spanMs >= 5 * 60 * 1000 ? 'mature' : 'seeding'; // Change 5 minutes
}
```

### Changing Snapshot Retention Window

In `src/app/api/nse/route.ts`:
```typescript
// Keep 20 minutes of history
priceSnapshots.filter(s => now - s.timestamp < 20 * 60 * 1000)
```

### Modifying the Port

In `package.json`:
```json
"dev": "next dev -p 3000 2>&1 | tee dev.log"
```

Change `3000` to your desired port. Also update `Caddyfile` accordingly.

---

## 13. Troubleshooting

### Problem: "Simulated Data" badge appears instead of Yahoo Finance

**Cause**: Yahoo Finance API may be blocked by network restrictions, or the server cannot reach `query1.finance.yahoo.com`.

**Solutions**:
1. Verify internet connectivity: `curl -I https://query1.finance.yahoo.com`
2. Check if a firewall or proxy is blocking outbound HTTPS requests
3. The app will automatically fall back to NSE India API, then simulated data
4. Yahoo Finance works best from non-Indian IPs (NSE India has geo-restrictions)

### Problem: All timeframe values show "N/A"

**Cause**: The application has just started and hasn't accumulated enough price snapshots.

**Solution**: Wait approximately 5 minutes. The 30-second values will appear first, followed by longer timeframes as history builds up. The maturity badge will change from "SEEDING" to "MATURE".

### Problem: NSE India API returns 403 Forbidden

**Cause**: NSE India actively blocks automated requests and requires specific headers and cookies.

**Solution**: This is expected. The Yahoo Finance API should be the primary source. If Yahoo is also unavailable, the app falls back to simulated data. No action needed — the fallback chain handles this automatically.

### Problem: Build fails with TypeScript errors

**Solution**: The project is configured with `ignoreBuildErrors: true` in `next.config.ts` and `noImplicitAny: false` in `tsconfig.json`. If you encounter build errors, ensure these settings are preserved. For development, use `bun run dev` which is more tolerant of type errors.

### Problem: High memory usage

**Cause**: In-memory storage of price snapshots for 435+ stocks with 20-minute retention.

**Solution**: Reduce the retention window or the stock universe:
- Reduce retention: Change `20 * 60 * 1000` to a smaller value in `route.ts`
- Reduce stocks: Remove symbols from `ALL_NSE_SYMBOLS` in `nse-api.ts`

### Problem: Yahoo Finance rate limiting

**Cause**: Too many requests in a short time.

**Solution**: The app already implements 200ms delays between batch requests. If you still hit rate limits, increase the delay in `nse-api.ts`:
```typescript
await delay(500); // Increase from 200ms to 500ms
```

---

## 14. Prompt for z.ai to Run This App

Copy and paste the following prompt to z.ai to have it set up and run this application:

---

```
I have an NSE Live Stock Tracker application built with Next.js 16, TypeScript, Tailwind CSS 4, and shadcn/ui. 
Please help me run it. Here is what you need to do:

1. The project is located at /home/z/my-project/ (or I will provide the path)
2. Run `bun install` to install all dependencies
3. Ensure the .env file has: DATABASE_URL=file:/home/z/my-project/db/custom.db
4. Run `bun run db:push` to set up the SQLite database
5. Run `bun run dev` to start the development server on port 3000
6. The app should be accessible at http://localhost:3000

Key features of the app:
- Real-time NSE stock tracking with 30-second auto-refresh
- Rolling interval percentage changes (30s, 1m, 2m, 3m, 4m, 5m)
- Volume analysis with trade signals (BUY/SELL/WATCH/WAIT), support/resistance, entry/SL/target
- Data sourced from Yahoo Finance API (primary), NSE India (fallback), or simulated (last resort)
- Dark theme dashboard with 4 tabs: Gainers, Losers, Analysis, Volume Analysis
- Data maturity indicators — "N/A" shown until 5 minutes of real data is collected

If Yahoo Finance API is unavailable from the server, the app will fall back to simulated data 
and show an amber "Simulated Data" warning banner. This is expected behavior.

The main files are:
- src/lib/nse-api.ts — Data fetching (Yahoo Finance + NSE India + simulated)
- src/app/api/nse/route.ts — Backend API with in-memory price tracking
- src/app/page.tsx — Frontend dashboard (1944 lines)

Please start the dev server and confirm the app is running.
```

---

## Quick Reference Card

| Action | Command |
|---|---|
| Install dependencies | `bun install` |
| Start dev server | `bun run dev` |
| Build for production | `bun run build` |
| Start production server | `bun run start` |
| Initialize database | `bun run db:push` |
| Run linter | `bun run lint` |
| Access dashboard | `http://localhost:3000` |
| Access via Caddy | `http://localhost:81` |
| API endpoint | `http://localhost:3000/api/nse?type=gainers&refresh=true` |

---

*This guide was generated for the NSE Live Tracker application. Last updated: April 2026.*
