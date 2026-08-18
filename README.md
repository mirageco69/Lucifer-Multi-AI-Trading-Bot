# LUCIFER — The AI Multi-Trading Platform

> **Version**: v2.1 &nbsp;|&nbsp; **Updated**: August 2026 &nbsp;|&nbsp; **Status**: Live Trading

LUCIFER is a self-hosted, plugin-based automated trading platform for cryptocurrency and CFD/commodity markets. It runs 24/7 on a dedicated Windows machine, trades across multiple exchanges simultaneously, and manages risk through a layered AI gate, strategy engine, and research pipeline.

---

## What It Does

| Capability | Detail |
|---|---|
| **Multi-exchange trading** | CoinSpot, Swyftx, Bybit (crypto) · IG Markets (CFDs/commodities) |
| **AI trading brain** | 7-agent autonomous brain — sole buy authority; intelligent exit debates consult ReasoningBank, MTF, regime, breadth before cutting a loss |
| **Strategy engine** | 139+ strategies across crypto and commodity asset classes |
| **Research pipeline** | Autonomous AI research agent, backtester, statistical tools, strategy certification |
| **Risk management** | Intelligent exit debates, trailing stops, DCA, drawdown limits, regime detection — no mechanical fixed stop-losses |
| **Dashboard** | HTTPS web dashboard — Mission Control, Charts, Research, Strategies, Settings |
| **Plugin system** | WordPress-style auto-discovery — 264+ active plugins, no restart to add new ones |
| **Telegram alerts** | Trade signals, daily P&L reports, AI decisions, breadth alerts |
| **Paper trading** | Full dry-run mode — real data, no real money |
| **IG Markets holdings** | Tiered commodity positions tracked separately with full P&L |

---

## Screenshots

### AI Brain Console
Live terminal feed of AI Brain decisions, debates, and executor activity. Colour-coded by action type — green for BUY, yellow for HOLD, red for EXIT, purple for DEBATE.

![AI Brain Console](https://raw.githubusercontent.com/mirageco69/Lucifer-Multi-AI-Trading-Bot/main/Images/AI%20Brain%20Console%201.png)

### AI Brain Widget
Mission Control Brain status widget — last decision, open positions, MTF gate status, and opportunity score at a glance.

![AI Brain Widget](https://raw.githubusercontent.com/mirageco69/Lucifer-Multi-AI-Trading-Bot/main/Images/AI%20Brain%20Widget.png)

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        LUCIFER Platform                         │
├────────────────┬────────────────┬──────────────────────────────┤
│   Trading      │   Research     │   Dashboard                   │
│   Engine       │   Pipeline     │   (React + Flask)             │
│                │                │                               │
│  bot.py        │  research/     │  dashboard-new/               │
│  services/     │  backtester/   │  src/plugins/*/widget.jsx     │
│  strategies/   │  optimize/     │  src/api/                     │
├────────────────┴────────────────┴──────────────────────────────┤
│                     Plugin System                               │
│  264+ plugins auto-discovered from src/plugins/**              │
│  Each plugin: plugin.json + api.py + widget.jsx                │
├────────────────────────────────────────────────────────────────┤
│              Exchanges                │   AI / LLM              │
│  CoinSpot · Swyftx · Bybit · IG      │  Ollama (local)         │
│  Paper exchange (dry-run)             │  Claude (cloud)         │
└────────────────────────────────────────────────────────────────┘
```

---

## Key Features

### Multi-Exchange Support
Trade the same strategy across CoinSpot (crypto, AUD), Swyftx (crypto, AUD), Bybit (crypto, USDT), and IG Markets (CFDs/commodities) simultaneously. Each exchange has its own plugin with independent lot tracking, P&L, and order management.

### 7-Agent Autonomous AI Brain
Every trade decision goes through a full multi-agent debate running on local Ollama (no cloud):
- **Technical Analyst** — RSI, MACD, EMA, multi-timeframe alignment
- **Sentiment Analyst** — Fear & Greed index, news, macro events
- **Fundamental Analyst** — portfolio exposure, cash, open positions
- **Bull Researcher** — strongest case for entering the trade
- **Bear Researcher** — strongest case against entering the trade
- **Risk Manager** (qwen2.5:14b) — final veto, ReasoningBank-informed, cannot be overruled
- **Portfolio Manager** — final position size and execution instruction

The brain is the **sole buy authority**. Bot signal-triggered buys are blocked when the Brain is enabled. Orders are queued to `brain_orders.json` and executed by a dedicated executor thread.

**Intelligent exit logic** replaces fixed stop-losses. When a position hits −3%, the Brain runs a 2-agent debate (Recovery Analyst + Risk Arbiter) consulting the ReasoningBank, trade history, MTF signal, regime, Fear & Greed, and market breadth before deciding to hold or cut. The floor tightens on each HOLD (−3% → −6% → −8% → −10%). Hard exits bypass debate: regime flip to BEAR, −10%, 7-day hold cap, +12% profit cap.

**Watchlist discovery** feeds the Brain's paper portfolio. The 72-coin watchlist's best candidates are evaluated by the Brain every 4 hours. BUY decisions go to a paper portfolio only. After 3 paper wins, a Telegram alert suggests promoting the coin to the live list.

**Research → Brain pipeline**: when the research agent promotes a strategy or the optimizer switches a coin's strategy, findings are automatically written to the ReasoningBank so the Brain consults them in future debates.

The Brain also learns — after every trade closes, a nightly reflection (2am) extracts patterns into the ReasoningBank.

### Strategy Engine (139+ Strategies)
Strategies are organised into two asset-class groups:

**Crypto Strategies** (RSI-based, momentum, mean reversion, ML, DCA variants, arbitrage, regime-aware)  
**Commodity Strategies** (trend following, carry/term structure, calendar spreads, relative value, seasonality, fundamentals)

Strategies can be individually enabled, backtested, hyperopt-tuned, and certified before deployment.

### Research Pipeline
- **Research Agent** — autonomous AI that analyses bot performance nightly (3am AEST), detects weaknesses, and generates research projects
- **Backtester** — per-strategy historical testing with configurable date ranges
- **Statistical Tools** — Hurst exponent, Monte Carlo simulation, rule significance testing
- **Strategy Certification** — formal pass/fail gate before any strategy reaches production
- **Strategy Lab** — 139-strategy bench for comparing performance across asset classes

### Plugin Doctor v2
A 3-LLM autonomous self-healing engine that runs every 5 minutes in the dashboard:
- Scans all 276+ plugins for missing files, import errors, and blueprint compliance gaps
- Diagnoses root cause via LLM, proposes and applies a fix
- Requests a targeted service restart via `data/restart_requests/`
- Protected files (bot.py, data/*.json, supervisor.ps1) are diagnosed but never auto-repaired

### Risk Management
- Per-trade stop-loss with configurable buffer
- Trailing stop-loss
- Fear-weighted DCA (buy more in Extreme Fear, less in Extreme Greed)
- Daily drawdown limits with automatic trading pause
- Position concentration limits
- Regime detection (trending / ranging / high-volatility)

### Paper Trading
Full paper-trading mode using real live market data. Switches to dry-run at the ribbon button — no code changes needed. All AI decisions, P&L tracking, and logs still run in real time.

---

## Workspaces (Dashboard Navigation)

| Workspace | Purpose |
|---|---|
| **Mission Control** | Portfolio overview, open positions, recent trades, exchange accounts |
| **Markets** | Live price feeds, charts, market breadth, sentiment snapshot |
| **Strategies** | Trading engine controls, strategy lab, optimisation, backtesting |
| **Research** | Research centre, AI agent, statistical tools, certification |
| **AI** | AI Brain console, reasoning bank, visual agent, LLM prompt editor, brain metrics |
| **Risk** | Risk framework, trailing stops, DCA rules, regime detector |
| **Exchange** | Per-exchange order books, trade history, ledger sync |
| **Settings** | All platform settings — trading, risk, AI, automation, notifications |
| **Engineering** | Blueprint health, tech debt, event bus, migration progress |

---

## Technology Stack

| Layer | Technology |
|---|---|
| **Backend** | Python 3.11+ · Flask · APScheduler |
| **Frontend** | React 18 · Vite · CSS Variables (5 themes) |
| **Database** | JSON flat-file store (data/) · SQLite for research |
| **AI** | Ollama (local LLMs) · Claude API (cloud, optional) |
| **Exchanges** | REST APIs — CoinSpot V2 · Swyftx JWT · Bybit V5 · IG Markets REST |
| **Hosting** | Self-hosted Windows · HTTPS on port 5443 · mkcert TLS |
| **Notifications** | Telegram Bot API |

---

## System Requirements

| Component | Minimum | Recommended |
|---|---|---|
| **OS** | Windows 10 64-bit | Windows 11 Pro 64-bit |
| **CPU** | Intel Core i5 (any gen) | Intel Core i7 / Ryzen 7 |
| **RAM** | 8 GB | 16 GB (required if running local LLMs) |
| **Storage** | 50 GB free | 1 TB SSD (data accumulates over time) |
| **Python** | 3.10+ | 3.11+ |
| **Node.js** | 18+ | 20 LTS |
| **Ollama** | Required for AI features | Latest version |
| **Network** | Broadband | Broadband with static or DDNS address for remote access |

**Local LLM models used**: `qwen3:8b` (fast — all real-time agents), `qwen2.5:14b` (deep — Risk Manager + reflection), `mistral-nemo:12b` (Plugin Doctor reviewer)

> A dedicated always-on machine is strongly recommended. The bot runs 24/7 and should not share resources with a primary workstation.

---

## Roadmap

### Completed
- [x] Multi-exchange trading (CoinSpot, Swyftx, Bybit, IG Markets)
- [x] 7-agent autonomous AI Brain — sole gatekeeper for all new opens
- [x] Self-learning ReasoningBank — grows from every closed trade
- [x] Nightly reflection engine (2am, qwen2.5:14b)
- [x] Position monitor — stop-loss / take-profit / max-hold exits
- [x] ATR-based + Kelly criterion position sizing
- [x] Multi-timeframe alignment gate (1h/4h/1d RSI+EMA)
- [x] Correlation gate (Pearson r > 0.85 blocks)
- [x] Regime-calibrated confidence thresholds
- [x] Agent performance tracker + debate weights
- [x] A/B testing framework for agent prompts (Thompson sampling)
- [x] Opportunity scanner (6-signal coin ranking every 5 min)
- [x] Brain metrics dashboard (equity curve, regime, agent stats, journal)
- [x] Daily Telegram digest (8am)
- [x] Smart alert manager (throttle, dedup, severity routing)
- [x] Plugin Doctor v2 (3-LLM autonomous self-healing)
- [x] 276+ strategy plugins with auto-discovery
- [x] Research centre with AI agent, backtester, statistical tools
- [x] Strategy certification pipeline
- [x] IG Markets tiered holdings
- [x] Fear-weighted DCA
- [x] LUCIFER rebrand
- [x] Mission Control movable/resizable widget cards
- [x] Live CoinSpot trading — enabled 2026-08-17
- [x] Brain as sole buy authority — bot signal buys blocked; Brain order queue + executor thread
- [x] Intelligent exit debates — 2-agent debate replaces fixed stop-loss (2026-08-18)
- [x] Watchlist MTF engine — yfinance removed; all timeframe signals from watchlist price array
- [x] Watchlist discovery pipeline — suggested coins paper-evaluated by Brain every 4h
- [x] IG Brain fully wired — IG-Brain + IG-BrainExecutor threads; brain_orders_ig.json queue
- [x] Research → Brain pipeline — research agent + optimizer write findings to ReasoningBank
- [x] Brain Console terminal widget — live decision feed on Mission Control

### In Progress
- [ ] Macro calendar auto-populate feed (macro gate currently inactive)
- [ ] Commodity strategy engines (trend, carry, seasonal, relative value)
- [ ] Brain `paper_only` flag flip to live (pending debate quality validation)

### Future
- [ ] Mobile app (Android — PWA wrapper + push notifications)
- [ ] Telegram breadth alerts
- [ ] A/B prompt experiment auto-bootstrap (framework built, experiments pending)
- [ ] Macro calendar feed (Gate 6 currently inactive)

---

## Licence

Private — all rights reserved. This software is not open source.

Contact: lucifersdevteam@gmail.com
