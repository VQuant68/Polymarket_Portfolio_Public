# 📈 VQuant - Polymarket BTC Up/Down Automated Trading Bot

> **An Institutional-Grade Automated Quantitative Trading Architecture bridging TradingView Technical Analytics with the Polymarket Decentralized Prediction Market Protocol on the Polygon Blockchain.**

---

## 🧭 1. Executive Summary & Business Case

The **Polymarket BTC Up/Down (5-Minute Cycle)** is a highly volatile derivative prediction market. Every 5 minutes, a new market resolves whether the price of Bitcoin will be higher or lower than its opening price. Trading this manually is practically impossible due to the sub-second execution speeds required to secure favorable ask prices before the liquidity pools saturate.

**VQuant Engineering** designed and deployed a comprehensive, fully automated pipeline to solve this. By integrating TradingView's advanced charting capabilities with a custom, high-throughput Python webhook gateway, the system programmatically detects highly probable setups on a micro-timeframe (2m) and injects digital signatures directly into Polymarket's Smart Contracts via the Polygon blockchain.

### 🎯 Key Objectives Achieved:
- **Zero-Latency Execution:** Capitalizing on the micro-spreads before the broader market reacts.
- **Emotionless Risk Mitigation:** Hard-coded position sizing and maximum fill-price ceilings.
- **Precision Timing:** Mathematical alignment with Polymarket's specific settlement epochs.

---

## 🏗️ 2. Core System Architecture (3-Tier Pipeline)

The system is decoupled into three distinct architectural layers to ensure maximum fault tolerance and processing speed.

```text
[TIER 1: ANALYTICS] TradingView (2m Chart) → Detects ultra-early signals (Volume Vector, MTF Trend, ADX).
       ▼ (Webhook Dispatch)
[TIER 2: GATEWAY]   Python Flask Server (Railway) → Handles Auth, Risk Management, Sizing & Security.
       ▼ (API Injection)
[TIER 3: EXECUTION] Polygon Blockchain / Polymarket RPC → FOK Market Order (Ask < 0.85).
       ▼ (On-Chain Settlement)
[LEDGER]            Transaction successful, outputs TxID to Cloud Logs.
```

---

## 📊 3. Product Showcase & Visual Proof

![VQuant Polymarket Bot Dashboard](Assets/dashboard_screenshot.jpg)
*Real-time TradingView Dashboard (VQuant Signal Engine) dynamically tracking Volume Vectors, MTF Trend alignments, and Timing Logic before dispatching execution signatures to the Polymarket Protocol.*

---

## 💻 4. Algorithmic Mechanics & Logic Deep Dive

### Phase 1: The Signal Engine (TradingView Pine Script v6)
The alpha-generation core is built in Pine Script v6. It prevents blind algorithmic trading by mathematically validating setups before firing webhooks. The script is fortified with institutional logic:

1. **Precision Epoch Timing (Modulo Logic)**
   - Polymarket cycles open every 5 minutes (e.g., `10:00`, `10:05`, `10:10`). To gain an edge, our script analyzes the **2-minute (2m) chart** but is mathematically locked to only trigger at the *exact start* of a 10-minute window.
   - Implements `minute(time, syminfo.timezone) % 10 == 0` to execute orders flawlessly at `00`, `10`, `20`, etc.
2. **Zero-Repaint Guarantee**
   - Enforces `barstate.isconfirmed == true`. The system only flags and dispatches signals when the candle cycle is perfectly frozen and written to history, preventing mid-candle false signals (repainting).
3. **Volume Vector Recognition**
   - Price action is meaningless without volume. The engine triggers *only* when the current candle's volume exceeds **200%** compared to the 10-candle Simple Moving Average (SMA).
4. **Momentum & Body Purity**
   - Filters out indecisive candles (Dojis, Spinning Tops). The real body of the candle must occupy `>60%` of the total high-low range.
5. **Macro-Trend Alignment (MTF)**
   - Built-in Multi-Timeframe (MTF) security requests ensure micro-signals align with the broader macroeconomic trend (`30m EMA > 1h EMA`).
   - Strictly implements `lookahead=barmerge.lookahead_off` to disable future data peaking (Anti-Future-Leak).

### Phase 2: Execution Gateway (Python / Railway Cloud)
A high-throughput API gateway built with Flask, acting as the defensive firewall between TradingView and the Blockchain.

1. **Dual-Cache Webhook Risk Firewall**
   - TradingView webhooks can occasionally double-fire due to network retries. The server hashes every incoming payload (`SHA-256(symbol+action+time)`) and caches it. Duplicate hashes within a 60-second window are instantly and silently dropped to protect capital.
2. **Polymarket On-Chain Execution**
   - Integrates the official `py-clob-client`.
   - Generates L2 signatures and interacts directly with the Polygon network to place **Fill-or-Kill (FOK)** orders.
3. **Fault Tolerance & RPC Auto-Healing**
   - The Polygon network is notorious for intermittent RPC congestion. The execution pipeline features a robust, non-blocking 2-Retry architecture with exponential backoff to bypass transient network failures.

---

## 🛡️ 5. Advanced Risk Management Framework

The bot does not just execute trades; it actively protects the portfolio utilizing strict, hard-coded financial parameters:

- **Dynamic Position Sizing (Auto-Compounding):** Before every trade, the bot queries the blockchain for the real-time USDC balance. It then aggressively controls exposure by sizing trades at exactly **2.5% of total available liquidity**. This allows the portfolio to compound during win streaks and mathematically prevents liquidation during drawdowns.
- **Absolute FOK Price Ceiling Limit:** Polymarket token prices range from $0.00 to $1.00. Buying too high destroys the Risk/Reward ratio. The bot intercepts the order book's Ask Price in milliseconds and **discards execution instantly if the token Ask Price exceeds $0.85**. 

---

## 📂 6. Portfolio Repository Structure

*(Note: Proprietary Pine Script algorithms, Python server source code, and private API credentials have been migrated to VQuant's Private Repositories. This public repository serves as an architectural showcase).*

```text
.
├── Assets/
│   └── dashboard_screenshot.jpg                 # UI & Proof of Execution
├── Reports/
│   └── VQuant_Phase1_Progress_Report.pdf        # Client handover documentation
├── polymarket_technical_spec_public.txt         # Distilled technical specifications
└── README.md                                    # Comprehensive System Architecture (You are here)
```

---

## 🚀 7. Infrastructure & Deployment Topology

The bot gateway was designed to be fully containerized, serverless, and highly available using Nixpacks on Railway.app.

```toml
# railway.toml (Infrastructure Build Specs)
[build]
builder = "nixpacks"

[deploy]
startCommand = "python app.py"
healthcheckPath = "/health"
healthcheckTimeout = 30
restartPolicyType = "always"
```
- **Security:** All Polygon Private Keys, Webhook Secrets, and Polymarket API Passphrases are injected via encrypted Cloud Environment Variables.
- **Logging Rotation:** Features a discrete System Logger module implementing cyclic overwrites (RotatingFileHandler), ensuring the cloud server disk never reaches 100% capacity from transaction logs.

---

*This repository is maintained by **VQuant Engineering**. It serves as an architectural portfolio piece demonstrating institutional-grade quantitative development, automated system integration, and Web3 infrastructure mastery.*
