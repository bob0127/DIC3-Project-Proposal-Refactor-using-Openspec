# Design: Hybrid Technical & Price Action Agent (Hybrid TPA-Agent)

## Context

Current quantitative trading approaches struggle with either lag and indicator noise or black-box predictions lacking spatial market structure awareness. Meanwhile, pure LLM-based financial agents suffer from severe numerical and spatial hallucinations when calculating exact price coordinates, stop-loss values, or boundary conditions. See `proposal.md - Why` for motivation.

The Hybrid TPA-Agent introduces a four-tier, eight-module decoupled architecture that assigns deterministic tasks to pure algorithms, probabilistic classification to machine learning, and semantic context validation to an LLM.

## Goals / Non-Goals

**Goals:**
- Provide a 100% deterministic spatial calculation engine for Swing Highs/Lows and Supply & Demand (S&D) zones without LLM hallucination.
- Integrate an XGBoost classifier to compute empirical breakout and bounce win probabilities based on spatial-momentum feature vectors.
- Establish a Chain-of-Thought (CoT) reasoning agent driven by Price Action domain knowledge rules to validate trade context and detect false breakouts.
- Enforce strict mathematical risk filters requiring $RR \ge 2.0$ and expected win rate $\ge 65\%$.
- Output machine-readable JSON and human-interpretable markdown reports for Human-in-the-Loop (HITL) execution.

**Non-Goals:**
- Ultra-low latency high-frequency trading (HFT) sub-second execution (the target operates on 15m to 1h intraday and swing horizons).
- Unattended fully autonomous trade order routing without human confirmation.
- Arbitrary cross-asset fundamental sentiment analysis (focus is purely on technical, spatial, and price action mechanics).

## Architecture & Module Breakdown

The system adopts a Tiered Block Architecture divided into 4 layers and 8 core sub-modules:

```
[Layer 1: Input & Data Layer]
  ├── [1.1] User Preference Block (Risk tolerances, capital, asset symbol)
  └── [1.2] Data Ingestion Engine (OHLCV ingestion & Volume Profile)
         │
         ▼
[Layer 2: Analytical Engines Layer]
  ├── [2.1] Deterministic Price Action & S&D Zone Engine (100% Python Math)
  └── [2.2] Machine Learning Win-Rate Engine (XGBoost Classifier)
         │
         ▼
[Layer 3: Reasoning & Knowledge Layer]
  ├── [3.1] Domain Knowledge Base (Price Action rules, S/R Flips, Liquidity)
  └── [3.2] Planner & Reasoning Agent (Chain-of-Thought LLM Synthesis)
         │
         ▼
[Layer 4: Risk & Output Layer]
  ├── [4.1] Decision & Risk Management Engine (Dynamic SL/TP, RR filtering)
  └── [4.2] Structured Report Interface (Actionable JSON/Markdown Setup)
```

### Module Descriptions
- **[1.1] User Preference Block**: Configures asset symbol (e.g. `BTCUSDT`), execution timeframe (`15m`, `1h`), max account risk per trade (e.g. 1-2%), and capital allocation.
- **[1.2] Data Ingestion Engine**: Connects to exchange APIs, fetches historical and live OHLCV bars, validates candle integrity, and derives Volume Profile price bins.
- **[2.1] Deterministic Price Action & S&D Zone Engine**: Calculates rolling swing extrema, consolidates overlapping pivot clusters, identifies strong impulse origin candles, and forms upper/lower boundaries for Demand and Supply Zones.
- **[2.2] Machine Learning Win-Rate Engine**: Extracts normalized technical features (RSI, ATR, Volume Z-score) paired with normalized geometric distances to nearest active zones, running XGBoost inference to predict bounce probability $P_{\text{bounce}}$.
- **[3.1] Domain Knowledge Base**: Encapsulates codified heuristics including Support-to-Resistance Flips (S/R Flips), liquidity pool absorption, Change of Character (ChoCh), and fake-out patterns.
- **[3.2] Planner & Reasoning Agent**: Consumes deterministic zone boundaries, ML probabilities, and knowledge prompts to reason about macro market context, rejecting conflicting or ambiguous setups.
- **[4.1] Decision & Risk Management Engine**: Determines dynamic Stop-Loss ($SL = \text{Zone}_{\text{bottom}} - 0.5 \times ATR$), Target Take-Profit ($TP = \text{Next Supply}_{\text{bottom}}$), checks $RR = \frac{TP - Entry}{Entry - SL} \ge 2.0$, and verifies positive expectancy $E = (P_{\text{win}} \times RR) - ((1 - P_{\text{win}}) \times 1) > 0$.
- **[4.2] Structured Report Interface**: Formats final setup into structured JSON payloads and clear, auditable trading briefs for trader one-click confirmation.

## Data Flow & Algorithm

### Execution Sequence
1. **Trigger**: System fetches new completed OHLCV bar for target asset and timeframe.
2. **Deterministic Geometry**: SNR Engine extracts swing points and identifies active Demand/Supply zones.
3. **ML Inference**: Features extracted from OHLCV and spatial distances; XGBoost evaluates $P_{\text{win}}$.
4. **Context Reasoner**: LLM evaluates structural context, checking if market structure confirms the ML signal.
5. **Risk Check**: Risk Engine computes $SL, TP, RR$; validates against $RR \ge 2.0$ and $P_{\text{win}} \ge 0.65$.
6. **Report Synthesis**: Approved setups or Wait/No-Trade reports are dispatched.

### Core Pseudocode
```python
Algorithm: Hybrid Technical & Price Action Agent (Hybrid TPA-Agent)
Input : Asset Symbol S, Timeframe TF, Historical OHLCV D
Output: Actionable Trading Setup & Decision Report R

01: raw_data = DataAcquisition.fetch_ohlcv(S, TF)
02: demand_zones, supply_zones = SNR_Engine.detect_zones(raw_data)
03: current_price = raw_data['close'].iloc[-1]
04: 
05: features = FeatureExtractor.extract(raw_data, demand_zones, supply_zones)
06: win_prob = ML_Engine.predict_win_rate(features)
07: 
08: agent_context = {
09:     "price": current_price,
10:     "demand_zones": demand_zones,
11:     "supply_zones": supply_zones,
12:     "win_probability": win_prob
13: }
14: evaluated_plan = PlannerAgent.reason(agent_context, PriceActionKnowledgeBase)
15: 
16: if evaluated_plan.win_prob >= 0.65 and evaluated_plan.rr_ratio >= 2.0:
17:     R = SynthesisEngine.generate_report(
18:         status="APPROVED",
19:         entry=current_price,
20:         stop_loss=evaluated_plan.stop_loss,
21:         take_profit=evaluated_plan.take_profit,
22:         evidence=evaluated_plan.evidence
23:     )
24: else:
25:     R = SynthesisEngine.generate_report(
26:         status="WAIT/NO_TRADE",
27:         reason="Risk-to-Reward (< 2.0) or Win Rate (< 0.65) below threshold."
28:     )
29: return R
```

## Data Contracts (JSON Schemas)

### Zone Detection Output
```json
{
  "symbol": "BTCUSDT",
  "timeframe": "1h",
  "current_price": 62450.0,
  "demand_zones": [
    {
      "zone_id": "DZ-1",
      "price_low": 60800.0,
      "price_high": 61250.0,
      "hvn_poc": 61050.0,
      "retest_count": 1,
      "strength": "strong"
    }
  ],
  "supply_zones": [
    {
      "zone_id": "SZ-1",
      "price_low": 64200.0,
      "price_high": 64650.0,
      "hvn_poc": 64400.0,
      "retest_count": 0,
      "strength": "moderate"
    }
  ]
}
```

### Final Decision Report Output
```json
{
  "status": "APPROVED",
  "timestamp": "2026-11-13T21:00:00Z",
  "symbol": "BTCUSDT",
  "action": "LONG",
  "entry_range": [61200.0, 61300.0],
  "stop_loss": 60550.0,
  "take_profit_1": 64200.0,
  "risk_reward_ratio": 4.15,
  "ml_win_probability": 0.72,
  "reasoning_summary": "Price successfully retested Demand Zone DZ-1 with long lower wick absorption and bullish ChoCh on 15m. Volume profile confirms institutional accumulation. Opposing supply zone SZ-1 provides ample asymmetric target.",
  "risk_disclaimer": "Maximum capital exposure: 1.5% of total balance."
}
```

## Decisions

- **Decoupling Math from LLM**: 
  - *Rationale*: LLMs are notoriously prone to floating-point math and spatial coordinate errors. Pure Python handles 100% of the geometry and indicators.
  - *Alternatives considered*: End-to-end prompt feeding raw OHLCV to multimodal LLMs (rejected due to hallucinations and latency).
- **XGBoost for Win-Rate Classification**:
  - *Rationale*: XGBoost excels on tabular numerical features with rapid inference time and resilience against overfitting compared to deep recurrent networks on modest dataset sizes.
  - *Alternatives considered*: LSTM/Transformers (higher compute requirements, slower, prone to overfitting non-stationary price distributions).
- **Human-in-the-Loop (HITL) Execution**:
  - *Rationale*: Eliminates catastrophe risk from sudden exchange API anomalies or flash-crashes. The trader retains final execution sign-off.
  - *Alternatives considered*: Fully automated API order submission.

## Risks / Trade-offs

- **[Risk: Market Regime Concept Drift]** → *Mitigation*: Continuous tracking of volume volatility metrics (ATR, Volume Z-score). When metrics exceed $3\sigma$, system pauses signal generation.
- **[Risk: Exchange API Latency or Disconnection]** → *Mitigation*: Three-tier fallback: Level 1 (heartbeat delay > 2s) stops new signals; Level 2 ensures server-side hard stop-loss is placed immediately upon entry; Level 3 (> 30s) triggers safety alert.
- **[Risk: High-Frequency Market Noise in Low Timeframes]** → *Mitigation*: Enforce multi-timeframe confirmation (1h zone baseline with 15m entry confirmation).
- **[Risk: Low Trade Frequency from Strict Filters]** → *Mitigation*: Accepted trade-off: Quality and positive mathematical expectancy take strict priority over trade volume.

