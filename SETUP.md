# LUCIFER — Setup & Installation Guide

> This guide covers a fresh installation of the LUCIFER trading platform on a dedicated Windows machine.

---

## System Requirements

| Component | Minimum | Recommended |
|---|---|---|
| **OS** | Windows 10 64-bit | Windows 11 Pro 64-bit |
| **CPU** | Intel Core i5 (any gen) | Intel Core i7 / AMD Ryzen 7 |
| **RAM** | 16 GB | **32 GB+** — recommended if running local LLMs |
| **Storage** | 250 GB free | 1 TB SSD — data accumulates over months |
| **GPU** | NVIDIA 8 GB VRAM | NVIDIA 8 GB+ VRAM — built and tested on RTX 5060 Ti |
| **Python** | 3.10+ | **3.11+** |
| **Node.js** | 18+ | **20 LTS** |
| **Ollama** | Latest | Latest |
| **Network** | Broadband | Broadband — static IP or DDNS recommended for remote access |

> A **dedicated always-on machine** is strongly recommended. LUCIFER runs 24/7 and should not share a primary workstation. Built and tested on Windows 11 Pro with an NVIDIA RTX 5060 Ti.

### GPU Notes

An NVIDIA GPU with at least 8 GB VRAM is the minimum recommended for running local LLMs at usable speed.

- **Without GPU**: LLMs run on CPU — expect 10–60 seconds per trade analysis
- **8 GB VRAM**: `qwen3:8b` fully on GPU — recommended minimum for the AI Brain
- **16 GB VRAM**: `qwen3:8b` + `qwen2.5:14b` both on GPU — full Brain capability
- **24 GB VRAM**: All models on GPU including Plugin Doctor reviewer (`mistral-nemo:12b`)

> NVIDIA (CUDA) only on Windows. AMD ROCm is experimental and untested.

---

## Prerequisites

### 1. Python 3.11+
Download from [python.org](https://python.org). During install, check **"Add Python to PATH"**.

```
python --version   # should show 3.11 or higher
```

### 2. Node.js 20 LTS
Download from [nodejs.org](https://nodejs.org). Choose the LTS release.

```
node --version     # should show v20.x
npm --version
```

### 3. Ollama (for local AI)
Download from [ollama.com](https://ollama.com) and install. Then pull the required models:

```
ollama pull qwen3:8b
ollama pull qwen2.5:14b
```

`qwen3:8b` is used for all real-time Brain agents. `qwen2.5:14b` is used for the Risk Manager agent and nightly reflection (deeper reasoning, runs overnight).

For the Plugin Doctor reviewer, also pull:
```
ollama pull mistral-nemo:12b
```

> All LLM inference is local — no cloud API calls are made. Ollama must be running before starting the bot.

### 4. mkcert (for HTTPS)
LUCIFER serves the dashboard over HTTPS. Install mkcert to generate a local trusted certificate.

Download `mkcert-v1.x.x-windows-amd64.exe` from the mkcert GitHub releases page. Rename it to `mkcert.exe` and place it in a folder on your PATH, then:

```
mkcert -install
```

This installs a local Certificate Authority so your browser trusts the self-signed cert.

---

## Installation

### Step 1 — Copy the Project

Place the project at `C:\projects\tradingbot\` (this path is expected by the supervisor and scripts).

### Step 2 — Create Python Virtual Environment

```
cd C:\projects\tradingbot
python -m venv venv_bot
venv_bot\Scripts\activate
pip install -r requirements.txt
```

### Step 3 — Install Dashboard Dependencies

```
cd dashboard-new
npm install
npm run build
```

The build output goes to `dashboard-new/dist/` and is served by Flask.

### Step 4 — Generate TLS Certificate

```
cd C:\projects\tradingbot\ssl
mkcert -cert-file cert.pem -key-file key.pem localhost 127.0.0.1 192.168.x.x
```

Replace `192.168.x.x` with your machine's LAN IP address. The dashboard will be available at `https://<your-ip>:5443`.

> On any other machine that will access the dashboard, you must also run `mkcert -install` to trust the local CA.

### Step 5 — Configure Environment

Copy `.env.example` to `.env` and fill in your credentials:

```ini
# Exchange API Keys
COINSPOT_API_KEY=your_key_here
COINSPOT_API_SECRET=your_secret_here

SWYFTX_API_KEY=your_key_here

BYBIT_API_KEY=your_key_here
BYBIT_API_SECRET=your_secret_here

IG_API_KEY=your_key_here
IG_USERNAME=your_username
IG_PASSWORD=your_password
IG_ACCOUNT_TYPE=DEMO   # or LIVE

# Dashboard
DASHBOARD_SECRET_KEY=generate_a_random_string_here

# Telegram (optional)
TELEGRAM_BOT_TOKEN=your_token
TELEGRAM_CHAT_ID=your_chat_id
```

> Never commit `.env` to version control. It is in `.gitignore`. The platform uses local LLMs only — no cloud AI API keys are required or used.

### Step 6 — Initial Settings

Edit `data/settings.json` to configure your trading parameters. At minimum:

```json
{
  "dry_run": true,
  "buying_enabled": false,
  "coinspot_active_coins": ["BTC", "ETH"],
  "trade_size_aud": 100,
  "min_trade_size_aud": 50
}
```

**Start with `dry_run: true` and `buying_enabled: false` until you have verified the bot is operating correctly.**

### Step 7 — Start the Platform

Run the supervisor script to start all processes:

```powershell
C:\projects\tradingbot\scripts\supervisor.ps1
```

The supervisor starts and monitors all configured processes, restarting them automatically if they crash. It is set up to run at Windows logon via Task Scheduler.

To restart all processes manually:

```powershell
C:\projects\tradingbot\scripts\restart_all.ps1
```

### Step 8 — Access the Dashboard

Open your browser and navigate to:
```
https://localhost:5443
```
or from another machine on your network:
```
https://192.168.x.x:5443
```

Log in with the admin credentials set during first-run setup.

---

## Auto-Start at Login (Supervisor)

The supervisor (`scripts/supervisor.ps1`) runs at Windows logon via Task Scheduler, starting and monitoring all configured processes without a password prompt.

> **Note**: The Store version of Python (from the Microsoft Store) does not support passwordless Task Scheduler tasks. Use the python.org installer.

---

## AI Brain

LUCIFER includes a 7-agent autonomous trading brain built entirely on local Ollama — no cloud API calls are ever made.

### How it works

When the Brain is enabled, it is the **sole buy authority**. The bot's own signal-triggered buys are blocked. Instead:

1. The Brain debate loop runs every 60 seconds
2. Six specialist agents debate the market context
3. A Risk Manager (using `qwen2.5:14b`) makes the final call
4. BUY decisions are queued to `data/brain_orders.json`
5. A dedicated executor thread picks up the queue every 15 seconds and places the order

### Exit logic

Fixed stop-losses are replaced with intelligent exit debates. When a position hits −3%, the Brain runs a 2-agent debate (Recovery Analyst + Risk Arbiter) before deciding to hold or cut. The floor tightens on each HOLD (−3% → −6% → −8% → −10%). Hard exits bypass debate: −10%, regime flip to BEAR, 7-day hold cap.

### Nightly reflection (ReasoningBank)

At **2am every night**, the Brain reviews all closed trades from the day and extracts patterns into the ReasoningBank — a local RAG memory consulted during every future debate. This runs for both crypto and IG Markets trades.

**Why 2am?**
- Crypto: markets are 24/7, so 2am is low-volatility and avoids interrupting active positions
- IG Markets (Australia): session closes ~10pm AEST. 2am gives 4 hours of closed-trade data to reflect on before the next open at 8am AEST

The reflection model (`qwen2.5:14b`) takes 5–15 minutes — this is expected. Ollama handles it in the background without affecting live trading.

### Watchlist discovery

Every 4 hours, the Brain paper-evaluates the best candidates from the 72-coin watchlist. BUY decisions go to a paper portfolio only. After 3 paper wins, a Telegram alert suggests promoting the coin to the live list.

### Enabling the Brain

In `data/settings.json` or from the dashboard (AI workspace → Brain Settings):

```json
{
  "ai_brain_enabled": true,
  "ai_brain_paper_only": true
}
```

Set `ai_brain_paper_only: false` only when you are satisfied with the quality of debates shown in the Brain Console (Mission Control → Brain Console widget).

---

## First-Time API Setup

### CoinSpot
1. Log in to CoinSpot → Account → API Keys
2. Create a new key with **Trading** permissions
3. Copy key and secret to `.env`

### Swyftx
1. Log in to Swyftx → Profile → API Management
2. Generate a new API key
3. Copy to `.env`

### Bybit
1. Log in to Bybit → Account & Security → API Management
2. Create key with **Trade** and **Read** permissions
3. Copy key and secret to `.env`

### IG Markets
1. Log in to IG → My IG → API
2. Create an API key
3. Use your IG username and password alongside the key in `.env`
4. Set `IG_ACCOUNT_TYPE=DEMO` to start — switch to `LIVE` only when ready

**IG trading hours (AEST):** Sunday 8am – Friday 10pm. The IG Brain automatically checks whether the session is open before placing any orders.

### Telegram Bot (Optional)
1. Message `@BotFather` on Telegram → `/newbot`
2. Copy the token to `.env`
3. Send a message to your bot, then find your chat ID from the Telegram API
4. Copy chat ID to `.env`

---

## Enabling Live Trading

LUCIFER ships with trading disabled for safety.

When you are confident the bot is behaving correctly in paper mode, enable live trading in `data/settings.json`:

```json
{
  "dry_run": false,
  "buying_enabled": true
}
```

Or toggle it from the dashboard: **Settings → Trading → Dry Run Mode**.

> ⚠️ Live trading with real money carries risk. The authors accept no responsibility for financial losses. Never risk more than you can afford to lose.

---

## Security Checklist

Before exposing the dashboard to the internet:

- [ ] Change default admin password (Settings → Security & 2FA)
- [ ] Enable 2FA (Settings → Security & 2FA → TOTP)
- [ ] Restrict `.env` file permissions (only your user should be able to read it)
- [ ] Do not commit `.env`, `data/settings.json`, or any file containing credentials
- [ ] Use a strong, random `DASHBOARD_SECRET_KEY`
- [ ] Consider a VPN rather than port-forwarding if possible
- [ ] Review `ssl/` — regenerate the cert if you change your IP or hostname

---

## Troubleshooting

### Dashboard won't load
Check logs at `logs/dashboard.log`. Common causes:
- Port 5443 already in use — stop any other process using that port
- Certificate mismatch — regenerate with `mkcert`
- Python import error on startup — run `pip install -r requirements.txt` again

### Bot not connecting to exchange
- Verify API keys are correct in `.env`
- Check exchange status pages for outages
- Look at `logs/bot_coinspot.log` for the specific error

### Ollama not responding
- Run `ollama serve` manually and check it starts
- Verify the models are pulled: `ollama list` — you need `qwen3:8b` and `qwen2.5:14b`
- Default endpoint is `http://localhost:11434` — check nothing else is on that port

### Brain Console shows no decisions
- The Brain only runs a full debate when the MTF gate passes (2 of 3 timeframes bullish)
- `WEAK_BULLISH (score +1/3)` in the LOG tab means it is waiting for better market alignment
- Check the LOG tab — `MTF gate BLOCKED` is normal during flat or bearish conditions

### Plugin not showing in dashboard
- Check `plugin.json` syntax (valid JSON, all required fields present)
- Run the Plugin Doctor from Settings to see specific errors
- Check browser console for import errors in the widget
- Rebuild the frontend: `npm run build` in `dashboard-new/`

### "Hyperopt service not available"
Optuna must be installed:
```
venv_bot\Scripts\pip install optuna
```

---

## Upgrade Notes

When updating LUCIFER:
1. Stop all running processes (`restart_all.ps1` or kill via supervisor)
2. Copy new files over the existing installation
3. Run `pip install -r requirements.txt` (dependencies may have changed)
4. Run `npm install && npm run build` in `dashboard-new/`
5. Restart the supervisor

Plugin files in `src/plugins/` are preserved across updates — your plugin customisations are not overwritten.

---

## Directory Structure

```
C:\projects\tradingbot\
  bot.py                    — Crypto trading runtime (CoinSpot, Swyftx, Bybit)
  ig_bot.py                 — IG Markets CFD trading runtime
  dashboard.py              — Flask application entry point
  services/                 — Shared Python services
    ai_brain_loop.py        — Autonomous Brain debate loop
    ai_decision_engine.py   — 7-agent debate pipeline
    exit_debate.py          — Intelligent exit debate engine
    reflection_engine.py    — Nightly ReasoningBank reflection (2am)
    reasoning_bank.py       — Local RAG memory for the Brain
    watchlist_mtf.py        — Multi-timeframe signals from watchlist data
    watchlist_discovery.py  — Discovery pipeline (paper evaluation of new coins)
  src/
    plugins/                — 278+ plugin folders (auto-discovered)
  dashboard-new/
    src/                    — React frontend source
    dist/                   — Built frontend (served by Flask)
  data/                     — Runtime data (JSON state files)
    brain_orders.json       — Live Brain buy queue (crypto)
    brain_orders_ig.json    — Live Brain buy queue (IG)
    exit_debates.json       — Open position exit debate state
    decision_ledger.json    — Full audit trail of all Brain decisions
    reasoning_bank.json     — Accumulated learned patterns
    watchlist_state.json    — 72-coin watchlist with price history
  logs/                     — Log files
    bot_coinspot.log        — Main crypto bot log
    ig_bot.log              — IG Markets bot log
    dashboard.log           — Dashboard log
  scripts/
    supervisor.ps1          — Process supervisor (auto-start at logon)
    restart_all.ps1         — Kill all processes and let supervisor restart
  ssl/
    cert.pem / key.pem      — TLS certificate
  docs/
    public/                 — This documentation
```

---

Contact: lucifersdevteam@gmail.com
