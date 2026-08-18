# LUCIFER Plugin Catalogue

> **264+ active plugins** · Auto-discovered at startup · No restart required to add a new plugin

LUCIFER uses a WordPress-style plugin architecture. Every feature is a plugin: exchanges, AI agents, data sources, indicators, strategies, risk rules, and dashboard widgets. Adding a plugin means dropping three files into `src/plugins/<category>/<name>/`:

```
plugin.json   — metadata (name, label, category, widget slot)
api.py        — Flask blueprint (backend routes)
widget.jsx    — React component (dashboard widget)
```

The platform discovers and loads them automatically. Disabling a plugin in the Plugin Manager removes it from the UI and API without touching any other code.

---

## Exchange Plugins

These plugins connect LUCIFER to trading exchanges. Each manages its own authentication, order placement, lot tracking, and P&L.

| Plugin | Exchange | Asset Class | Notes |
|---|---|---|---|
| `coinspot` | CoinSpot | Crypto (AUD) | V2 REST API · Market + limit orders |
| `swyftx` | Swyftx | Crypto (AUD) | JWT auth · Full order management |
| `bybit` | Bybit | Crypto (USDT) | V5 API · Spot trading |
| `ig` / `ig_exchange` | IG Markets | CFDs / Commodities | REST API · Tiered holdings |
| `paper_exchange` | Paper Trading | All asset classes | Simulates all exchanges with real market data |
| `binance_exchange` | Binance | Crypto | Stub — not yet enabled |
| `kraken_exchange` | Kraken | Crypto | Stub — not yet enabled |
| `ib_exchange` | Interactive Brokers | Equities / Futures | Stub — not yet enabled |

---

## AI Plugins

| Plugin | Purpose |
|---|---|
| `ollama` | Core Ollama integration — routes LLM calls to local models |
| `local_ollama_advisor` | Pre-trade gate — validates every trade with structured LLM reasoning |
| `claude_advisor` | Cloud AI gate — disabled (zero cloud API policy; local Ollama only) |
| `ensemble` | Ensemble voting — combines multiple strategy signals into a weighted decision |
| `ai_chat_assistant` | In-dashboard AI chat assistant |
| `llm_prompts` | **LLM Prompt Editor** — edit all AI system prompts from the dashboard without touching Python files. Changes take effect immediately, no restart. |
| `mc_ai_brain` | Mission Control — AI Brain status widget (last decision, open positions, MTF gate, opportunity) |
| `brain_console` | **Brain Console** — live terminal feed of AI Brain decisions, debates, executor activity, and exit debate state. Tabs: All / Decisions / Log / Queue. |

### LLM Prompt Keys (editable from dashboard)

| Key | Agent | Group |
|---|---|---|
| `prebuy_filter` | Legacy single-agent pre-buy filter | Legacy |
| `agent_technical` | Technical analysis agent | 4-Agent Panel |
| `agent_sentiment` | Sentiment analysis agent | 4-Agent Panel |
| `agent_risk` | Risk management agent | 4-Agent Panel |
| `agent_vision` | Vision/chart agent | 4-Agent Panel |
| `pretrade_gate` | Final pre-trade approval gate | AI Gate |
| `visual_agent` | Dashboard Visual Agent page | Tools |

---

## Data Source Plugins

| Plugin | Data Provided |
|---|---|
| `fear_greed` | Crypto Fear & Greed Index (Alternative.me) |
| `crypto_news` | Latest crypto news aggregated from multiple feeds |
| `rss_crypto_news` | RSS-based crypto news parser |
| `macro_news` | Macroeconomic and geopolitical news (MACRO/ tagged) |
| `sentiment_snapshot` | Point-in-time market sentiment composite |
| `reddit_sentiment` | Reddit crypto subreddit sentiment scoring |
| `influencer_feed` | Crypto influencer post monitoring |
| `coingecko_intel` | CoinGecko market intelligence — trending, top gainers/losers (one bulk call per cycle, not per-coin) |
| `watchlist_mtf` | Watchlist-derived multi-timeframe signal — aggregates 15-min price array into 1h/4h/6h bars. Replaces yfinance. No external API calls. |
| `market_chart` | OHLC price data and chart feeds |
| `deribit_feed` | Deribit options data — IV, put/call ratio |
| `derivatives_feed` | Futures funding rates and open interest |
| `onchain_macro_feed` | On-chain macro indicators (whale movements, exchange flows) |
| `system_health_monitor` | CPU, RAM, disk, process health monitoring |

---

## Indicator Plugins

These plugins compute technical indicators on price data and make them available to strategies.

| Plugin | Indicator |
|---|---|
| `rsi` | Relative Strength Index |
| `ema` | Exponential Moving Average (configurable periods) |
| `sma` | Simple Moving Average |
| `macd` | MACD line, signal line, histogram |
| `bollinger_bands` | Bollinger Bands with configurable stddev |
| `atr` | Average True Range |
| `adx` | Average Directional Index + DI+/DI- |
| `supertrend` | SuperTrend directional signal |
| `stochastic_rsi` | Stochastic RSI oscillator |
| `ichimoku` | Ichimoku Cloud (Tenkan, Kijun, Senkou A/B, Chikou) |
| `vwap` | Volume Weighted Average Price |
| `obv` | On-Balance Volume |
| `swing_failure_pattern` | SFP detection (liquidity sweep reversal) |
| `functional` | Utility indicator helpers |

---

## Risk Rule Plugins

| Plugin | Function |
|---|---|
| `core` | Core risk rules — exposure limits, daily drawdown, position count caps |
| `fear_dca` | Fear-weighted DCA — increases buy size in Extreme Fear, reduces in Extreme Greed |
| `dca` | Standard DCA rules — dollar-cost averaging on dips |
| `trailing_stop` | Trailing stop-loss with configurable trail percentage |
| `sell_decision` | Lot-based sell decision engine — FIFO P&L, take-profit targeting |

---

## Notification Plugins

| Plugin | Channel |
|---|---|
| `telegram` / `telegram_notification` | Telegram Bot — trade alerts, daily reports, AI decisions, 2FA codes |
| `webhook_notification` | Generic webhook — POST trade events to any endpoint |

---

## Service Plugins

| Plugin | Function |
|---|---|
| `exchange_ledger_sync` | Syncs executed trades from exchange APIs into the local ledger |

---

## Strategy Plugins — Crypto (100+)

Strategies are grouped into logical families. All can be individually enabled, backtested, hyperopt-tuned, and certified.

### Momentum & Trend Following
`adx_momentum_strategy` · `adx_trend_strategy` · `ema_crossover_strategy` · `ema_crossover_long_strategy` · `ema_crossover_medium_strategy` · `ema_scalp_strategy` · `donchian_strategy` · `donchian_turtle_strategy` · `breakout_strategy` · `atr_breakout_strategy` · `supertrend_strategy` · `elder_ray_strategy` · `momentum_breakout_strategy` · `momentum_oscillator_strategy` · `trend_following_strategy` · `multi_timeframe_trend_strategy`

### Mean Reversion & Oscillators
`bollinger_strategy` · `bollinger_breakout_strategy` · `bollinger_scalp_strategy` · `bollinger_squeeze_strategy` · `rsi_strategy` · `rsi_divergence_strategy` · `rsi_reversal_strategy` · `stochastic_rsi_strategy` · `cci_strategy` · `awesome_oscillator_strategy` · `mean_reversion_strategy` · `swing_failure_pattern_strategy`

### DCA & Position Building
`dca_strategy` · `dca_smart_strategy` · `dca_value_averaging_strategy` · `anti_martingale_strategy` · `equal_weight_rebalance_strategy`

### AI & Machine Learning
`ai_strategy` · `ensemble_voting_strategy` · `anomaly_detection_strategy` · `neural_network_strategy` · `reinforcement_learning_strategy` · `black_litterman_strategy` · `copula_strategy` · `entropy_strategy`

### Regime-Aware
`regime_strategy` · `drawdown_regime_strategy` · `correlation_regime_strategy` · `volatility_regime_strategy` · `market_regime_strategy` · `adaptive_regime_strategy`

### Arbitrage & Rotation
`exchange_arb_strategy` · `dex_arb_strategy` · `dominance_rotation_strategy` · `sector_rotation_strategy` · `multi_asset_rotation_strategy`

### Advanced & SMC (Smart Money Concepts)
`breaker_block_strategy` · `equal_highs_lows_strategy` · `order_block_strategy` · `liquidity_sweep_strategy` · `fair_value_gap_strategy` · `wyckoff_accumulation_strategy` · `multi_timeframe_strategy` · `cointegration_strategy`

### Statistical & Research
`hurst_exponent_strategy` · `monte_carlo_strategy` · `rule_significance_strategy` · `z_score_strategy` · `pairs_trading_strategy`

*...and 50+ additional crypto strategy variants*

---

## Strategy Plugins — Commodity / CFD (30+)

Purpose-built for IG Markets CFD trading across energies, metals, grains, and soft commodities.

### Trend Following
`cfd_trend_following_strategy` · `commodity_momentum_rotation_strategy` · `cfd_breakout_retest_strategy`

### Mean Reversion & Range
`cfd_mean_reversion_strategy` · `cfd_range_strategy` · `cfd_scalp_strategy`

### Carry & Term Structure
`cfd_carry_trade_strategy` · `commodity_term_structure_strategy`

### Calendar Spreads
`commodity_calendar_spread_strategy` · `seasonal_roll_strategy`

### Relative Value
`cfd_correlation_strategy` · `commodity_relative_value_strategy` · `inter_commodity_spread_strategy`

### Seasonality
`commodity_seasonality_strategy` · `agricultural_seasonal_strategy` · `energy_seasonal_strategy`

### Fundamental / Macro
`commodity_fundamentals_strategy` · `supply_demand_strategy` · `cfd_news_straddle_strategy`

### Risk Management for CFDs
`cfd_hedging_strategy` · `cfd_gap_trading_strategy`

---

## Plugin Doctor

The Plugin Doctor is a built-in diagnostics tool accessible from the Settings workspace. It scans all installed plugins and reports:

- Plugins missing required files (`plugin.json`, `api.py`, `widget.jsx`)
- Category mismatches between `plugin.json` and folder location
- Blueprint compliance gaps (missing required fields)
- Import errors caught at startup
- Plugins disabled vs enabled count

Warnings are colour-coded: **red** for errors that will prevent a plugin working, **amber** for compliance warnings, **green** for passing checks.

---

## Adding a New Plugin

1. Create `src/plugins/<category>/<plugin_name>/`
2. Add `plugin.json` (see template below)
3. Add `api.py` with a Flask Blueprint at `/api/<plugin_name>`
4. Add `widget.jsx` exporting a named component matching `widget.component`
5. Restart the dashboard (or hot-reload if in dev mode)

```json
{
  "name": "my_plugin",
  "label": "My Plugin",
  "description": "What this plugin does.",
  "version": "1.0.0",
  "author": "LUCIFER",
  "enabled": true,
  "certified": false,
  "category": "data_sources",
  "widget": {
    "component": "MyPluginWidget",
    "slot": "data-my-plugin",
    "file": "widget.jsx"
  }
}
```

The platform picks it up automatically — no registration required.
