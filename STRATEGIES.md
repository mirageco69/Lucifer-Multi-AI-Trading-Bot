# LUCIFER Strategy Catalogue

> **139 strategy plugins** across crypto and CFD/commodity asset classes · All backtestable · All hyperopt-tunable · All certifiable

---

## How Strategies Work

Every strategy in LUCIFER is a self-contained plugin. A strategy plugin:
- Receives indicator data (RSI, MACD, EMA, ATR, Bollinger, volume, Fear & Greed, etc.)
- Returns a signal: **BUY**, **SELL**, or **HOLD**
- Is independently backtestable against historical data
- Can be hyperopt-tuned with Optuna to find optimal parameters
- Must pass Strategy Certification before being deployed to live/paper trading

Strategies are selected from the Trading Engine controls page. Only one strategy is active at a time per exchange (or the ensemble voting strategy can combine multiple).

### Strategy Lifecycle

```
Created as plugin → Backtested → Statistical significance check →
Monte Carlo stress test → Strategy Certification → Strategy Lab (paper) →
Promoted to production
```

---

## Crypto Strategies

### Momentum & Trend Following

| Strategy | Description |
|---|---|
| `adx_momentum_strategy` | Uses ADX to confirm trend strength (>25) before entering in the direction of DI+/DI-. Avoids whipsaws in ranging markets. |
| `adx_trend_strategy` | ADX-based trend following with EMA confirmation. Enters on ADX breakout above threshold with price above EMA. |
| `ema_crossover_strategy` | Classic EMA crossover — fast EMA crossing above slow EMA triggers BUY; crossing below triggers SELL. Default: EMA9 / EMA21. |
| `ema_crossover_long_strategy` | Longer-period EMA crossover for swing trading. Uses EMA21/EMA50 for fewer but higher-conviction signals. |
| `ema_crossover_medium_strategy` | Medium-term EMA crossover. Balances signal frequency and noise reduction. |
| `ema_scalp_strategy` | Fast EMA crossover for short-term scalping. Uses EMA5/EMA13. Higher frequency, tighter stops. |
| `donchian_strategy` | Buys on breakout above the Donchian Channel upper band; sells on break below lower band. Trend-following with clear entry/exit levels. |
| `donchian_turtle_strategy` | Implements the classic Turtle Trading rules using Donchian Channels with a 20-period entry and 10-period exit. |
| `breakout_strategy` | Price breakout from a consolidation range. Detects when price breaks above recent highs with volume confirmation. |
| `atr_breakout_strategy` | ATR-based breakout. Enters when price moves ATR × multiplier above the recent range high. Adapts to volatility. |
| `supertrend_strategy` | Uses the SuperTrend indicator (ATR-based) to signal direction changes. Enters on direction flip. |
| `supertrend_momentum_scalper_strategy` | Combines SuperTrend direction with RSI momentum for higher-frequency scalp entries within the trend. |
| `supertrend_pullback_strategy` | Waits for a pullback within a SuperTrend uptrend before entering, improving entry price. |
| `elder_ray_strategy` | Elder Ray Index (Bull Power / Bear Power) to measure buying and selling pressure relative to an EMA. |
| `momentum_strategy` | Rate-of-change based momentum. Buys when momentum crosses above zero; sells when it crosses below. |
| `momentum_rotation_strategy` | Rotates between coins based on which has the strongest momentum score over a lookback period. |
| `momentum_regime_strategy` | Momentum strategy that adapts parameters based on the detected market regime. |
| `multi_tf_trend_strategy` | Multi-timeframe trend confirmation. Requires trend alignment across two or more timeframes before entry. |
| `trend_following_strategy` | Pure trend-following using moving average slope and ADX. Simple, robust, and easy to understand. |
| `trend_strength_regime_strategy` | Detects regime by measuring trend strength, then applies appropriate trend-following or range parameters. |

### Mean Reversion & Oscillators

| Strategy | Description |
|---|---|
| `bollinger_strategy` | Buys on touch of lower Bollinger Band with RSI oversold confirmation; sells on upper band touch. |
| `bollinger_breakout_strategy` | Buys on breakout above the upper Bollinger Band (expansion signal); opposite for shorts. |
| `bollinger_scalp_strategy` | Fast mean-reversion scalp between Bollinger Band middle and outer bands. |
| `bollinger_squeeze_strategy` | Detects Bollinger Band squeeze (very tight bands) as a precursor to volatility expansion and trades the breakout. |
| `keltner_bollinger_squeeze_strategy` | Squeeze detection using both Bollinger Bands and Keltner Channels. When BB is inside KC, squeeze is active — trade the expansion. |
| `keltner_reversion_strategy` | Mean reversion using Keltner Channel outer bands as overbought/oversold levels. |
| `rsi_strategy` | Core RSI strategy. Buys when RSI crosses above 30 (oversold recovery); sells when RSI crosses below 70. |
| `rsi_momentum_strategy` | RSI combined with momentum filter. Enters on oversold RSI only when momentum is also turning up. |
| `rsi_scalp_strategy` | Fast RSI scalp — tighter thresholds (35/65), shorter period RSI for higher-frequency signals. |
| `rsi_divergence_strategy` | Detects RSI divergence (price makes new high but RSI doesn't) as a reversal signal. |
| `stoch_rsi_strategy` | Stochastic RSI oscillator — faster than RSI, catches short-term overbought/oversold extremes. |
| `stoch_rsi_macd_strategy` | Combines Stochastic RSI oversold signal with MACD bullish crossover for higher-conviction entries. |
| `cci_strategy` | Commodity Channel Index strategy. Buys on CCI crossing above -100 from oversold; sells on CCI crossing below +100. |
| `awesome_oscillator_strategy` | Bill Williams' Awesome Oscillator — momentum strategy using the difference between two SMAs of midpoint prices. |
| `mean_reversion_strategy` | Statistical mean reversion using Z-score of price relative to rolling mean. Enters when price is significantly above/below mean. |
| `z_score_reversion_strategy` | Z-score based mean reversion with configurable entry threshold (e.g. Z > 2.0 = overbought). |
| `mfi_momentum_strategy` | Money Flow Index — volume-weighted RSI. Identifies money flowing into or out of an asset. |
| `williams_r_strategy` | Williams %R oscillator. Identifies overbought/oversold conditions with faster response than RSI. |
| `tsi_strategy` | True Strength Index — double-smoothed momentum oscillator for trend direction and momentum changes. |
| `rate_of_change_divergence_strategy` | ROC divergence detection — price and rate-of-change diverging signals exhaustion. |
| `parabolic_sar_strategy` | Parabolic SAR for trailing stop and trend direction. Flips signal when price crosses the SAR dots. |
| `macd_strategy` | MACD line crossing the signal line as a buy/sell trigger with histogram direction filter. |
| `macd_histogram_strategy` | Trades based on MACD histogram direction changes rather than line crossovers for earlier signals. |
| `macd_zero_line_strategy` | MACD crossing the zero line — a stronger trend signal than the signal line crossover. |
| `pivot_point_strategy` | Uses daily/weekly pivot points as support/resistance levels for entry and exit decisions. |
| `fibonacci_retracement_strategy` | Identifies Fibonacci retracement levels (38.2%, 50%, 61.8%) as entry zones within a trend. |
| `vwap_strategy` | VWAP as a dynamic mean-reversion anchor. Buys on dips below VWAP in uptrend; sells on rallies above in downtrend. |
| `vwap_rsi_strategy` | VWAP combined with RSI confirmation for higher-conviction mean-reversion entries. |
| `vwap_scalp_strategy` | Fast VWAP scalp — entries on brief deviations from VWAP that quickly snap back. |

### DCA & Position Building

| Strategy | Description |
|---|---|
| `dca_strategy` | Standard dollar-cost averaging. Buys additional lots at configurable price drop intervals. |
| `dca_smart_strategy` | Smart DCA — adjusts buy amounts based on how far price has fallen and current Fear & Greed level. |
| `dca_value_averaging_strategy` | Value averaging variant — targets a specific portfolio growth rate rather than fixed buy intervals. |
| `anti_martingale_strategy` | Anti-Martingale position sizing — increases bet size after wins, reduces after losses. Opposite of Martingale. |
| `martingale_limited_strategy` | Limited Martingale — doubles position after a loss up to a configurable maximum multiplier cap. High risk; requires careful limits. |
| `kelly_criterion_strategy` | Kelly Criterion position sizing — sizes each trade to maximise logarithmic growth based on estimated win probability and payoff ratio. |
| `equal_weight_rebalance_strategy` | Periodically rebalances portfolio back to equal weights across all active coins. |
| `risk_parity_strategy` | Allocates capital so that each position contributes equally to overall portfolio risk (measured by volatility). |
| `max_sharpe_strategy` | Optimises portfolio allocation to maximise the Sharpe ratio (risk-adjusted return). |
| `mean_variance_strategy` | Markowitz mean-variance portfolio optimisation — maximises expected return for a given risk level. |
| `min_volatility_strategy` | Portfolio allocation targeting minimum overall volatility, regardless of expected return. |
| `black_litterman_strategy` | Black-Litterman model — combines market equilibrium returns with analyst views to produce portfolio weights. |

### Grid Trading

| Strategy | Description |
|---|---|
| `grid_strategy` | Classic grid trading — places buy and sell orders at fixed price intervals above and below current price. Profits from oscillation. |
| `grid_dynamic_strategy` | Dynamic grid — adjusts grid spacing based on ATR (wider in high volatility, tighter in low volatility). |
| `grid_trailing_strategy` | Trailing grid — the entire grid moves up as price rises, locking in gains. |

### AI & Machine Learning

| Strategy | Description |
|---|---|
| `ai_strategy` | Uses the local LLM (via Ollama) to generate a BUY/SELL/HOLD signal based on a structured market data prompt. The AI model is the strategy. |
| `ensemble_voting_strategy` | Weighted ensemble of multiple strategy signals. Each strategy's recent win rate determines its vote weight. |
| `anomaly_detection_strategy` | Detects statistical anomalies in price or volume — unusual deviations that may signal an impending move. |
| `random_forest_signal_strategy` | Random Forest classifier trained on historical indicator data to predict next-candle direction. |
| `xgboost_strategy` | XGBoost gradient boosting model for price direction prediction. Trained on feature vectors from technical indicators. |
| `lstm_prediction_strategy` | LSTM neural network for sequence-based price prediction. Captures temporal dependencies in price data. |
| `genetic_algorithm_strategy` | Uses a genetic algorithm to evolve strategy parameters, selecting for win rate and Sharpe ratio over generations. |
| `reinforcement_learning_strategy` | Reinforcement learning agent that learns an optimal trading policy through simulated reward signals. |
| `sentiment_nlp_strategy` | NLP-based sentiment strategy. Scores recent news headlines and social media text to generate a directional bias. |
| `social_volume_strategy` | Trades based on spikes in social media volume for a coin — high social volume often precedes price movement. |
| `on_chain_strategy` | Uses on-chain data (whale movements, exchange inflows/outflows) as trading signals. |
| `whale_alert_strategy` | Monitors large transaction alerts (whale moves) and trades in the direction of significant capital flows. |

### Regime-Aware

| Strategy | Description |
|---|---|
| `regime_adaptive_strategy` | Detects the current market regime and switches between a trend-following and mean-reversion sub-strategy accordingly. |
| `regime_hmm_strategy` | Hidden Markov Model regime detection — identifies latent market states (bull/bear/sideways) and adapts strategy. |
| `drawdown_regime_strategy` | Adjusts position sizing and entry thresholds based on current drawdown level. Conservative in drawdown, aggressive in recovery. |
| `correlation_regime_strategy` | Uses correlation between assets as a regime indicator. High correlation = risk-on; breakdown = risk-off. |
| `volatility_regime_strategy` | Adapts to volatility regime — uses mean reversion in low-volatility periods and trend following in high-volatility. |
| `high_volatility_regime_strategy` | Specialised for high-volatility periods. Wider stops, smaller positions, faster exits. |
| `low_volatility_accumulation_strategy` | Accumulates positions during low-volatility consolidation phases before an expected expansion. |
| `momentum_regime_strategy` | Momentum strategy that scales position size and aggressiveness based on detected regime. |
| `seasonal_regime_strategy` | Incorporates known seasonal patterns (e.g. Q4 crypto rallies, summer doldrums) into regime classification. |
| `macro_regime_strategy` | Uses macroeconomic indicators (Fed policy, DXY, risk-on/off signals) to set the regime for crypto trading. |

### Smart Money Concepts (SMC)

| Strategy | Description |
|---|---|
| `breaker_block_strategy` | Identifies breaker blocks — former support that became resistance (or vice versa) — as high-probability re-entry zones. |
| `order_block_strategy` | Detects institutional order blocks — price zones where large orders were placed, acting as future support/resistance. |
| `liquidity_sweep_strategy` | Detects liquidity sweeps (stop hunts above highs or below lows) and trades the reversal. |
| `fair_value_gap_strategy` | Identifies Fair Value Gaps (FVG) — imbalances between candles — and enters when price returns to fill the gap. |
| `equal_highs_lows_strategy` | Detects equal highs/lows (liquidity pools) and anticipates the sweep-and-reverse pattern. |
| `market_structure_strategy` | Tracks market structure shifts — break of structure (BOS) and change of character (CHoCH) as trend signals. |
| `institutional_candle_strategy` | Identifies unusually large institutional candles as signals of significant order flow direction. |
| `smart_money_divergence_strategy` | Divergence between price and inferred smart money positioning as a reversal signal. |
| `supply_demand_zone_strategy` | Maps supply and demand zones from historical order flow and enters on retests of those zones. |

### Arbitrage & Rotation

| Strategy | Description |
|---|---|
| `exchange_arb_strategy` | Cross-exchange arbitrage — detects price discrepancies between CoinSpot, Swyftx, and Bybit and trades the spread. Note: CoinSpot fees are 0.01%/side — round-trip must exceed 0.02% to be profitable. |
| `dex_arb_strategy` | DEX vs CEX arbitrage — monitors decentralised exchange prices vs centralised exchange for discrepancies. |
| `futures_spot_arb_strategy` | Cash-and-carry arbitrage — exploits the difference between spot price and futures price. |
| `funding_rate_arb_strategy` | Exploits funding rate differentials on perpetual futures across exchanges. |
| `funding_sentiment_strategy` | Uses funding rates as a sentiment signal — extreme positive funding = crowded long = potential reversal. |
| `triangular_arb_strategy` | Three-leg arbitrage within a single exchange across multiple trading pairs. |
| `latency_arb_strategy` | Latency arbitrage — exploits millisecond price lag between data feeds. Requires very low-latency setup. |
| `dominance_rotation_strategy` | Rotates capital between BTC and altcoins based on BTC dominance index. High dominance = hold BTC; falling dominance = rotate to alts. |
| `pairs_trading_strategy` | Statistical pairs trading — finds two historically correlated coins and trades the divergence, expecting reversion. |
| `cointegration_strategy` | Tests for cointegration between coin pairs and trades deviations from the long-run equilibrium. |
| `stat_arb_strategy` | Statistical arbitrage using z-score of spread between cointegrated pairs. |

### Swing Trading

| Strategy | Description |
|---|---|
| `swing_strategy` | Core swing strategy — identifies swing highs and lows to enter near support and exit near resistance. |
| `swing_ema_pullback_strategy` | Swing entries on pullbacks to the EMA in an established trend. |
| `swing_channel_strategy` | Trades within price channels — buys near the lower channel boundary, sells near the upper. |
| `swing_rsi_divergence_strategy` | Swing strategy triggered by RSI divergence at swing points. |
| `swing_support_resistance_strategy` | Identifies and trades from key horizontal support and resistance levels. |
| `swing_ichimoku_strategy` | Ichimoku-based swing strategy — uses cloud support/resistance and Tenkan/Kijun crosses for entries. |
| `swing_macd_weekly_strategy` | Longer-term swing strategy using weekly MACD signals for position entries. |
| `swing_volume_breakout_strategy` | Swing breakout confirmed by volume spike above average. |

### Scalping

| Strategy | Description |
|---|---|
| `microstructure_scalp_strategy` | Scalps based on order book microstructure — bid/ask imbalance, trade velocity, and tick direction. |
| `order_flow_scalp_strategy` | Order flow scalping — uses trade prints and aggressiveness (market buys vs sells) as directional signal. |
| `spread_scalp_strategy` | Scalps the bid-ask spread on illiquid or wide-spread markets. |
| `tick_scalp_strategy` | Extremely short-term scalp based on tick-level price movement patterns. |
| `short_scalp_strategy` | Short-side scalp strategy for bear markets or overbought conditions. |
| `session_open_momentum_strategy` | Trades the momentum surge that occurs at major session opens (Asia, London, New York). |
| `opening_range_breakout_strategy` | Defines the opening range (first 30/60 minutes of session) and trades breakouts from that range. |
| `news_event_breakout_strategy` | Trades volatility breakouts around major news events (CPI, FOMC, crypto regulatory announcements). |

### Statistical & Research

| Strategy | Description |
|---|---|
| `hurst_exponent_strategy` | Calculates the Hurst exponent to determine whether a series is trending (H>0.5), mean-reverting (H<0.5), or random (H≈0.5). Selects appropriate sub-strategy accordingly. |
| `entropy_strategy` | Uses Shannon entropy of price returns as a regime indicator. Low entropy = trending; high entropy = random/ranging. |
| `garch_volatility_strategy` | GARCH model for volatility forecasting. Positions are sized based on predicted near-term volatility. |
| `fourier_cycle_strategy` | Fourier decomposition to identify dominant price cycles and trade cyclical turns. |
| `kalman_filter_strategy` | Kalman filter smoothing for noise reduction and dynamic parameter estimation. |
| `copula_strategy` | Statistical copula model to measure tail dependence between asset pairs for joint probability signals. |

### Fear & Greed Based

| Strategy | Description |
|---|---|
| `fear_greed_strategy` | Trades directly based on Fear & Greed extremes. Buys Extreme Fear (≤25); sells Extreme Greed (≥76). Pure contrarian. |
| `flash_crash_buy_strategy` | Detects sudden sharp price drops (flash crash signature) and enters a buy with tight stop. High reward if it recovers; defined risk if not. |
| `liquidation_cascade_strategy` | Identifies liquidation cascade signatures (rapid volume + price drop) and positions for the snapback. |

---

## CFD / Commodity Strategies (IG Markets)

These strategies are purpose-built for IG Markets CFD trading across energies (crude oil, natural gas), metals (gold, silver, copper), agricultural commodities (corn, wheat, coffee, cocoa, sugar), and indices.

### Trend Following

| Strategy | Description |
|---|---|
| `cfd_trend_following_strategy` | Core trend-following strategy for commodities — uses EMA crossovers and ADX to trade established trends. Wider stops to handle commodity volatility. |
| `commodity_momentum_rotation_strategy` | Rotates between commodity sectors based on momentum rankings. Holds the top-performing sector, exits the underperformer. |
| `cfd_breakout_retest_strategy` | Identifies a key breakout level, waits for a retest of that level, then enters on confirmation. Higher probability than chasing the initial breakout. |

### Mean Reversion & Range

| Strategy | Description |
|---|---|
| `cfd_mean_reversion_strategy` | Mean reversion for ranging commodity markets using Bollinger Bands and RSI. Works best in low-volatility, range-bound periods. |
| `cfd_range_strategy` | Identifies established price ranges and trades between the high and low boundaries of the range. |
| `cfd_scalp_strategy` | Short-term CFD scalp — tighter profit targets and stops suited to the spread structure of CFD instruments. |

### Carry & Term Structure

| Strategy | Description |
|---|---|
| `cfd_carry_trade_strategy` | Exploits the carry differential — long commodities in backwardation (futures cheaper than spot), short those in contango. |

### Relative Value & Correlation

| Strategy | Description |
|---|---|
| `cfd_correlation_strategy` | Trades correlated commodity pairs (e.g. corn/wheat, crude oil/natural gas) — long the underperformer, short the outperformer when divergence occurs. |

### Hedging & Risk Management

| Strategy | Description |
|---|---|
| `cfd_hedging_strategy` | Hedges existing crypto exposure using inversely correlated CFD positions. |
| `cfd_gap_trading_strategy` | Trades gap fills — when a CFD opens with a significant gap from the previous close, fades the gap expecting it to fill. |

### News & Events

| Strategy | Description |
|---|---|
| `cfd_news_straddle_strategy` | Places positions on both sides of a major commodity report (EIA, USDA, OPEC) to profit from the volatility spike regardless of direction. |

---

## Strategy Comparison Tools

All strategies in LUCIFER can be:

| Tool | What It Does |
|---|---|
| **Backtester** | Run any strategy against historical OHLC data for any coin/instrument and date range |
| **Hyperopt** | Use Optuna to search for optimal parameter values (RSI period, EMA periods, ATR multiplier, etc.) |
| **Monte Carlo** | Generate 1,000+ randomised equity curve simulations to stress-test expected performance |
| **Rule Significance** | Statistical test to verify that a strategy's rules have genuine predictive power vs random noise |
| **Strategy Lab** | Side-by-side comparison of all 139 strategies by win rate, Sharpe, max drawdown, trade count |
| **Certification** | Formal pass/fail gate — a strategy must pass before it can go live |

---

## Adding a New Strategy

1. Create `src/plugins/strategies/<strategy_name>/`
2. Add `plugin.json` with `"category": "strategies"`
3. Add `api.py` with your Flask Blueprint
4. Add `widget.jsx` for any dashboard display
5. Implement the strategy logic (typically in a `strategy.py` file imported by `api.py`)

A strategy receives indicator values and returns `{"signal": "BUY" | "SELL" | "HOLD", "confidence": 0-100, "reason": "..."}`.

Contact lucifersdevteam@gmail.com to discuss contributing a new strategy.
