# LUCIFER — Setup & Installation Guide

> Full guide for installing LUCIFER on a dedicated Windows machine from scratch.

---

## Before You Start

LUCIFER is a 24/7 trading platform. It is not a script you run occasionally — it runs continuously, manages real money (or simulated money in paper mode), and makes time-sensitive decisions. For best results:

- Use a **dedicated machine** that stays on permanently. Don't share it with your main workstation.
- Start in **paper trading mode** and watch the bot's behaviour for at least a week before enabling live trading.
- Read the Security section before exposing the dashboard to the internet.
- Never share your `.env` file or API keys with anyone.

---

## System Requirements

| Component | Minimum | Recommended |
|---|---|---|
| **OS** | Windows 10 64-bit | Windows 11 Pro 64-bit |
| **CPU** | Intel Core i5 (any gen) | Intel Core i7 / AMD Ryzen 7 |
| **RAM** | 8 GB | **16 GB** — required if running local LLMs |
| **Storage** | 50 GB free | 1 TB HDD/SSD — data accumulates over months |
| **GPU** | Not required | NVIDIA GPU with 8 GB+ VRAM recommended for local LLMs |
| **Python** | 3.10+ | **3.11+** (strongly recommended) |
| **Node.js** | 18+ | **20 LTS** |
| **Ollama** | Required for AI gate | Latest stable release |
| **Network** | Broadband | Broadband with static IP or DDNS for remote access |

> **Note**: All production testing has been done on Windows 11 Pro with an Intel Core i7, 16 GB RAM, and a 1 TB HDD. That is the reference configuration.

### GPU Notes

A GPU is **not required** — LUCIFER will run without one. However, it matters for the local AI components:

- **Without GPU**: Local LLMs (llama3.1:8b, phi3:mini) run on CPU. Expect 10–60 second response times per trade analysis depending on your CPU. The bot compensates with a longer AI timeout.
- **With GPU (NVIDIA 8 GB+ VRAM)**: Response times drop to 1–5 seconds. Enables the Visual Agent (qwen3-coder:30.5b at ~18 GB requires 24 GB VRAM or CPU offload).
- **VRAM recommendations**:
  - 8 GB VRAM → `phi3:mini` runs fully on GPU (fast), `llama3.1:8b` offloads some layers
  - 12 GB VRAM → `llama3.1:8b` runs fully on GPU
  - 24 GB VRAM → full Visual Agent on GPU

> NVIDIA cards only — Ollama on Windows supports CUDA. AMD GPU (ROCm) is experimental and not tested. CPU inference works on any machine.

---

## Prerequisites

Install these before touching the LUCIFER files.

### 1. Python 3.11+

Download from **python.org** — do **not** use the Microsoft Store version (it cannot run passwordless scheduled tasks, which the supervisor requires).

During installation:
- ✅ Check **"Add Python to PATH"**
- ✅ Check **"Install for all users"** (if prompted)

Verify:
```
python --version
# Should show: Python 3.11.x or higher
```

### 2. Node.js 20 LTS

Download from **nodejs.org** — choose the **LTS** release, not the Current release.

Verify:
```
node --version    # Should show: v20.x.x
npm --version     # Should show: 10.x.x or higher
```

### 3. Ollama (Local AI)

Download from **ollama.com** and install. Ollama runs as a background service on `http://localhost:11434`.

After installing, pull the required models:

```
ollama pull llama3.1:8b
```

This is the primary pre-trade gate model (~4.7 GB download).

```
ollama pull phi3:mini
```

This is the fast fallback model used when `llama3.1:8b` times out (~2.2 GB).

For the Visual Agent (chart screenshot analysis — optional):
```
ollama pull qwen3-coder:30.5b
```

> This model is large (~18 GB). Only pull it if you plan to use the Visual Agent feature.

Verify Ollama is running:
```
ollama list
# Should show the pulled models
```

### 4. mkcert (HTTPS Certificate)

LUCIFER serves the dashboard over HTTPS. You need a locally trusted certificate.

1. Download `mkcert-v1.x.x-windows-amd64.exe` from the mkcert GitHub releases page
2. Rename it to `mkcert.exe`
3. Place it in a folder on your PATH (e.g. `C:\Windows\System32\` or a custom tools folder)
4. Run:

```
mkcert -install
```

This installs a local Certificate Authority that your browser will trust.

> On any **other** machine that will access the dashboard over the network, you also need to run `mkcert -install` on that machine to trust the local CA — otherwise the browser will show a certificate warning.

### 5. Git (Optional)

Only needed if you are receiving updates via git. Download from **git-scm.com**.

---

## Installation

### Step 1 — Place the Project

Copy or clone the project to:
```
C:\projects\tradingbot\
```

This path is expected by the supervisor script and internal references. Using a different path requires updating `scripts/supervisor.ps1`.

### Step 2 — Python Virtual Environment

Open PowerShell and run:

```powershell
cd C:\projects\tradingbot
python -m venv venv_bot
venv_bot\Scripts\activate
pip install -r requirements.txt
```

This creates an isolated Python environment. All dependencies install into `venv_bot\` — nothing touches your system Python.

If you see errors during `pip install`, try:
```powershell
pip install --upgrade pip
pip install -r requirements.txt
```

### Step 3 — Install Dashboard Dependencies

```powershell
cd C:\projects\tradingbot\dashboard-new
npm install
npm run build
```

The build output goes to `dashboard-new/dist/`. Flask serves this as the dashboard frontend.

> First build may take 2–5 minutes — npm downloads all frontend dependencies.

### Step 4 — Generate the TLS Certificate

```powershell
cd C:\projects\tradingbot\ssl
mkcert -cert-file cert.pem -key-file key.pem localhost 127.0.0.1 192.168.x.x
```

Replace `192.168.x.x` with your machine's actual LAN IP address. You can find it with:
```
ipconfig
```
Look for "IPv4 Address" under your active network adapter.

The dashboard will be accessible at:
- `https://localhost:5443` (on the bot machine itself)
- `https://192.168.x.x:5443` (from other machines on your network)

### Step 5 — Configure Your Environment

Copy the environment template:
```powershell
copy .env.example .env
```

Open `.env` in a text editor and fill in your credentials:

```ini
# ── Exchange API Keys ──────────────────────────────────────────────

# CoinSpot (Australian crypto exchange)
COINSPOT_API_KEY=your_coinspot_api_key
COINSPOT_API_SECRET=your_coinspot_api_secret

# Swyftx (Australian crypto exchange)
SWYFTX_API_KEY=your_swyftx_api_key

# Bybit (international crypto exchange)
BYBIT_API_KEY=your_bybit_api_key
BYBIT_API_SECRET=your_bybit_api_secret

# IG Markets (CFDs and commodities)
IG_API_KEY=your_ig_api_key
IG_USERNAME=your_ig_username
IG_PASSWORD=your_ig_password
IG_ACCOUNT_TYPE=DEMO        # Use DEMO until you are ready for live trading

# ── Dashboard ─────────────────────────────────────────────────────

# Generate a long random string — this signs JWT tokens
DASHBOARD_SECRET_KEY=change_this_to_a_long_random_string_at_least_32_chars

# ── Telegram (Optional but recommended) ──────────────────────────

TELEGRAM_BOT_TOKEN=your_telegram_bot_token
TELEGRAM_CHAT_ID=your_telegram_chat_id

# ── Claude API (Optional — for cloud AI gate) ────────────────────

ANTHROPIC_API_KEY=your_claude_api_key
```

> ⚠️ **Never commit `.env` to version control.** It is in `.gitignore`. Treat it like a password.

### Step 6 — Initial Settings

Edit `data/settings.json` with your starting configuration. If this file doesn't exist, create it:

```json
{
  "dry_run": true,
  "buying_enabled": false,
  "coinspot_active_coins": ["BTC", "ETH"],
  "swyftx_active_coins": ["BTC", "ETH"],
  "trade_size_aud": 100,
  "min_trade_size_aud": 50,
  "max_daily_drawdown_pct": 5.0,
  "stop_loss_pct": 3.0,
  "trailing_stop_pct": 2.0
}
```

**Keep `dry_run: true` and `buying_enabled: false` until you have watched the bot running in paper mode and are satisfied with its behaviour.**

### Step 7 — First-Time Admin Account

On first run, the dashboard will prompt you to create an admin account. Set a strong password and save it somewhere secure. You can enable 2FA after logging in.

### Step 8 — Test Launch

Before setting up auto-start, run everything manually first to verify it works:

**Terminal 1 — Start the bot:**
```powershell
cd C:\projects\tradingbot
venv_bot\Scripts\python.exe bot.py
```

**Terminal 2 — Start the dashboard:**
```powershell
cd C:\projects\tradingbot
venv_bot\Scripts\python.exe src\app.py
```

Open your browser and go to `https://localhost:5443`. You should see the LUCIFER dashboard login page.

If everything loads correctly, stop both processes (Ctrl+C) and proceed to Step 9.

### Step 9 — Set Up Auto-Start (Supervisor)

The supervisor script starts and monitors all processes automatically when Windows logs in.

Open PowerShell as Administrator and run:

```powershell
schtasks /create /tn "LUCIFER Supervisor" /tr "powershell.exe -NonInteractive -WindowStyle Hidden -File C:\projects\tradingbot\scripts\supervisor.ps1" /sc onlogon /rl highest /f
```

This creates a Task Scheduler task that:
- Runs at login (no password prompt required)
- Runs with highest privileges
- Starts `supervisor.ps1` which launches and monitors all processes

To verify the task was created:
```powershell
schtasks /query /tn "LUCIFER Supervisor"
```

**Log out and back in** to test that the supervisor starts automatically. Then check `https://localhost:5443` — the dashboard should be up without you manually starting anything.

Supervisor logs are written to `logs/supervisor.log`.

---

## Exchange API Setup

### CoinSpot
1. Log in to CoinSpot → **Account** → **API Keys**
2. Click **Create New Key**
3. Enable **Trading** permissions (read + trade; do not enable withdrawal)
4. Copy the **API Key** and **API Secret** to `.env`

### Swyftx
1. Log in to Swyftx → **Profile** (top right) → **API Management**
2. Click **Generate API Key**
3. Copy the key to `.env`

### Bybit
1. Log in to Bybit → **Account & Security** → **API Management**
2. Click **Create New Key**
3. Enable **Read** and **Trade** permissions. Do not enable withdrawal.
4. Set an IP whitelist if possible (use your bot machine's public IP)
5. Copy **API Key** and **Secret Key** to `.env`

### IG Markets
1. Log in to IG → **My IG** (top right) → **Manage API Keys**
2. Create a new API key
3. Copy the key to `.env`
4. Use your IG username and password as `IG_USERNAME` and `IG_PASSWORD`
5. Set `IG_ACCOUNT_TYPE=DEMO` initially

> IG's DEMO account uses a separate environment with virtual funds. It is ideal for testing. When you switch to `LIVE`, real money is used.

### Telegram Bot Setup
1. Open Telegram and search for **@BotFather**
2. Send `/newbot` and follow the prompts
3. Choose a name and username for your bot
4. Copy the **token** BotFather gives you to `TELEGRAM_BOT_TOKEN`
5. Start your new bot (search for it and press Start)
6. Send it any message, then visit:
   `https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates`
7. Find the `chat.id` value in the response — copy it to `TELEGRAM_CHAT_ID`

---

## Configuring Telegram Alerts

Once your bot token and chat ID are in `.env`, go to **Settings → System → Telegram** in the dashboard to configure which alerts fire:

| Alert | Recommended Setting |
|---|---|
| Trade executed | ✅ On |
| AI gate blocked | ✅ On |
| Daily P&L report | ✅ On (set your preferred time) |
| Stop-loss triggered | ✅ On |
| System health warning | ✅ On |
| Bot started / stopped | ✅ On |
| 2FA codes | ✅ On (required for 2FA login) |

Test by clicking **Send Test Message** in the dashboard.

---

## Enabling Live Trading

LUCIFER ships with all trading disabled. The path to live trading:

**Step 1 — Paper trade for at least a week**
Watch the bot's decisions in paper mode. Check the Reasoning Bank daily — are the AI decisions sensible? Is the strategy performing as expected?

**Step 2 — Review the risk settings**
In Settings → Risk, confirm:
- Stop-loss percentage is appropriate for your risk tolerance
- Daily drawdown limit is set
- Position size is what you intended
- Exposure limits are conservative

**Step 3 — Enable trading**

In `data/settings.json`:
```json
{
  "dry_run": false,
  "buying_enabled": true
}
```

Or from the dashboard: **Settings → Trading → Dry Run Mode → switch off**.

Or from the ribbon bar at the top of the dashboard: click **Paper Trading** to toggle live mode.

> ⚠️ Live trading involves real money and real risk. The LUCIFER team accepts no responsibility for financial losses. Only trade with money you can afford to lose. Start with small trade sizes and increase gradually as you gain confidence in the system's behaviour.

---

## Running the Dashboard in Dev Mode

If you are developing features for LUCIFER, you can run the frontend in hot-reload mode:

```powershell
# Terminal 1 — backend
cd C:\projects\tradingbot
venv_bot\Scripts\python.exe src\app.py

# Terminal 2 — frontend dev server (hot reload)
cd C:\projects\tradingbot\dashboard-new
npm run dev
```

The frontend dev server runs on port 5173. It proxies API calls to the Flask backend on port 5443. Any change to `.jsx` files reloads instantly without rebuilding.

---

## Security Checklist

Work through this before exposing the dashboard to the internet:

- [ ] **Change the admin password** — Settings → Security & 2FA → Change Password
- [ ] **Enable 2FA** — Settings → Security & 2FA → TOTP. Scan the QR code with Google Authenticator or similar. 2FA codes are also sent to Telegram.
- [ ] **Use a strong `DASHBOARD_SECRET_KEY`** — at least 32 random characters. Generate one with: `python -c "import secrets; print(secrets.token_hex(32))"`
- [ ] **Restrict `.env` file permissions** — right-click → Properties → Security → remove all users except your own account
- [ ] **Do not commit `.env`** — verify it is listed in `.gitignore`
- [ ] **Review TLS cert** — regenerate the cert if you change your IP or machine name
- [ ] **Consider a VPN** rather than direct port-forwarding for remote access
- [ ] **IG account type** — double-check `IG_ACCOUNT_TYPE` is `DEMO` until you are deliberately going live
- [ ] **Review API key permissions** — all exchange API keys should have **trade** access only, never withdrawal

---

## Troubleshooting

### Dashboard won't load
1. Check `logs/dashboard.log` for errors
2. Verify port 5443 is not in use: `netstat -aon | findstr :5443`
3. If a certificate error in browser — open `https://localhost:5443`, click Advanced → Proceed anyway. If it persists, regenerate the cert with `mkcert`.
4. If a Python import error — run `venv_bot\Scripts\pip install -r requirements.txt` again

### Bot not connecting to an exchange
1. Check the specific exchange log: `logs/bot_coinspot.log`, `logs/bot_swyftx.log`, etc.
2. Verify API keys are correct in `.env` — no extra spaces or newlines
3. Check the exchange's status page for outages
4. For IG: confirm `IG_ACCOUNT_TYPE` matches the account type you have (DEMO vs LIVE)

### Ollama not responding
1. Open PowerShell and run `ollama serve` — check it starts without errors
2. Verify models are pulled: `ollama list`
3. Check nothing else is on port 11434: `netstat -aon | findstr :11434`
4. Restart the Ollama service from Windows Services if needed

### Plugin not appearing in dashboard
1. Check `plugin.json` is valid JSON (paste it into a JSON validator)
2. Check the category field matches the folder location
3. Run Plugin Doctor from Settings — it shows the specific error
4. Check the browser developer console (F12) for JavaScript import errors
5. Restart the dashboard

### Hyperopt service not available
Install Optuna:
```powershell
venv_bot\Scripts\pip install optuna
```

### Supervisor not starting processes at login
1. Open Task Scheduler → find "LUCIFER Supervisor" → check it ran (Last Run Result should be 0x0)
2. Check `logs/supervisor.log` for errors
3. Verify the PowerShell path in the task matches your actual PowerShell location
4. Ensure you are using the python.org Python, not the Microsoft Store version

### "Port 5443 already in use"
A previous instance is still running. Find and kill it:
```powershell
netstat -aon | findstr :5443
taskkill /PID <the_pid_shown> /F
```

---

## Updating LUCIFER

1. Stop all processes (supervisor, bot, dashboard)
2. Copy in the new files (or pull with git)
3. Reinstall dependencies:
   ```powershell
   venv_bot\Scripts\pip install -r requirements.txt
   ```
4. Rebuild the dashboard:
   ```powershell
   cd dashboard-new
   npm install
   npm run build
   ```
5. Restart the supervisor

Plugin files in `src/plugins/` are never overwritten by updates — your custom plugins and any edits to existing plugins survive the update.

---

## Directory Structure

```
C:\projects\tradingbot\
│
├── bot.py                        Main trading runtime
├── requirements.txt              Python dependencies
├── .env                          Your credentials (never commit this)
│
├── src\
│   ├── app.py                    Flask application entry point
│   ├── api\                      Flask blueprints (core API routes)
│   └── plugins\                  264+ plugin folders
│       ├── exchanges\
│       ├── ai\
│       ├── data_sources\
│       ├── indicators\
│       ├── strategies\
│       ├── risk_rules\
│       ├── notifications\
│       └── services\
│
├── dashboard-new\
│   ├── src\                      React frontend source
│   └── dist\                     Built frontend (served by Flask)
│
├── services\                     Shared Python service modules
├── data\                         Runtime data (JSON state files, settings)
├── logs\                         Log files (bot, dashboard, supervisor)
│
├── scripts\
│   └── supervisor.ps1            Process supervisor — auto-start at login
│
├── ssl\
│   ├── cert.pem                  TLS certificate
│   └── key.pem                   TLS private key
│
└── docs\
    └── public\                   This documentation
```

---

## Getting Help

- Email: **lucifersdevteam@gmail.com**
- See [README.md](README.md) for platform overview
- See [PLUGINS.md](PLUGINS.md) for plugin catalogue
- See [STRATEGIES.md](STRATEGIES.md) for all 139 strategy descriptions
