# Polymarket BTC 15m Assistant [Volatility Gate]
### Stage 3: Volatility Gate Alone (<45 bp 1h ATR)

A real-time, high-performance automated trading bot for Polymarket **"Bitcoin Up or Down" 15-minute** binary options markets, executing via **Good 'Til Date (GTD)** limit orders powered by **Stage 3: Apex Engine + Volatility Gate** from the optimization report.

---

## 🏆 Empirical Backtest Reference (83.4% Win Rate)

Evaluated across **historical Polymarket BTC 15-minute markets**:
- **Strategy**: Stage 3 — Volatility Gate Alone (<45 bp 1h ATR)
- **Win Rate**: **83.4%** (2,215 executed trades)
- **Profit Factor**: **1.56**
- **Annualized Sharpe Ratio**: **4.95**
- **Max Drawdown**: **-51.4%**

---

## 🧠 Hardcoded Quantitative Strategy Architecture

### 1. The Trailing 1-Hour Realized Volatility Gate (`entry_block_reason`)
- **Condition**: Automatically pauses trading whenever trailing 1-hour realized ATR exceeds **45.0 basis points** over the last 4 15-minute candles:
  $$\text{vol\_1h\_bp} = \frac{\max(\text{highs}[-4:]) - \min(\text{lows}[-4:])}{\text{latest\_close}} \times 10,000 \le 45.0\text{ bp}$$
- **Purpose**: Eliminates losses caused by macro news spikes (CPI, FOMC), ETF rebalances, and exchange liquidation cascades where violent late-window whipsaws breach strike prices.
- **Normal Volatility Regime**: When $\text{vol\_1h\_bp} \le 45.0\text{ bp}$, order book conditions are calm, unlocking an **83.4% win rate**.

### 2. Alpha & Entry Rules (`decide_ev`)
- **Continuous EV Latency Arbitrage Engine**: Evaluates closed-form Geometric Brownian Motion (GBM) fair probability from live Binance spot ticks against Polymarket CLOB book best asks ($EV \ge 0.04$, min prob $0.55$).
- **Window Timing Envelope ($90\text{s} \le t \le 600\text{s}$)**:
  - **Skips the first 90 seconds (Minute 0 noise)**: Bypasses strike price discovery and wide bid/ask spreads.
  - **Skips after Minute 10 ($t > 600\text{s}$)**: Bypasses the late-candle decay trap where winning contracts inflate to 85¢–95¢ and offer poor risk/reward.
- **Strike Moneyness Margin of Safety (`min_strike_distance_bp: 6.0`)**:
  - Requires Bitcoin to have moved at least **6.0 basis points** ($0.06\% \approx \$60$ on BTC) away from the opening strike in the direction of the bet:
    - **UP**: Spot must be $\ge +6.0\text{ bp}$ above strike.
    - **DOWN**: Spot must be $\le -6.0\text{ bp}$ below strike.
  - Eliminates the 50/50 near-the-money coin-flip trap.
- **15m Structural Heikin-Ashi Alignment (`use_15m_ha: True`)**:
  - Prohibits betting DOWN when 15m HA has a flat bottom (bullish institutional momentum).
  - Prohibits betting UP when 15m HA has a flat top (bearish institutional momentum).
- **Micro-Vetoes DISABLED (`use_vetoes: false`)**.

---

## ⚡ GTD Low-Latency Execution Engine

- **Order Type**: Good 'Til Date (`OrderType.GTD`) limit orders with a **5-minute expiration timestamp**.
- **Signing Speed**: Sub-0.1ms C-accelerated EIP-712 signing via `coincurve`.
- **Pre-Warmed Token Caches**: Pre-fetched token IDs for instant zero-REST execution.
- **Reactive L2 WebSocket Feed**: Instant event-driven trade evaluation.
- **Default Port**: **`8086`**.

---

## 🏃 Running the Bot

```powershell
python main.py
```

Dashboard access:
👉 **`http://localhost:8086`**
