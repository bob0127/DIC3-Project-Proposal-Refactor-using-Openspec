# Tasks: Hybrid Technical & Price Action Agent (Hybrid TPA-Agent)

## 1. Environment & Data Pipeline Setup

- [ ] 1.1 Set up Python project environment, verify dependencies (`pandas`, `numpy`, `xgboost`, `scikit-learn`, `pydantic`) install cleanly
- [ ] 1.2 Implement Data Ingestion Engine for fetching historical OHLCV data from CCXT or Binance API and verify data integrity with automated schema tests
- [ ] 1.3 Implement horizontal Volume Profile computation with binning and verify calculation accuracy against test benchmark data

## 2. Deterministic Price Action & S&D Zone Engine

- [ ] 2.1 Implement rolling window Swing High and Swing Low detection algorithm and verify pivot detection across historical sample data
- [ ] 2.2 Implement Supply and Demand Zone identification logic based on price consolidation and expansion candles, and verify zone boundary outputs
- [ ] 2.3 Implement Support/Resistance flip (S/R Flip) tracking logic when zones are breached, and verify state transitions with unit tests

## 3. Machine Learning Win-Rate Prediction Engine

- [ ] 3.1 Implement spatial and technical feature extraction pipeline (ATR, RSI, Volume Z-Score, distance to nearest zones) and verify feature matrix shape and bounds
- [ ] 3.2 Train and calibrate XGBoost classification model on labeled historical bounce/breakout dataset and verify AUC-ROC and F1 metrics exceed baseline
- [ ] 3.3 Build real-time ML inference wrapper that outputs probability scores and verify sub-50ms inference latency

## 4. LLM Reasoning Agent & Domain Knowledge Base

- [ ] 4.1 Construct Price Action Domain Knowledge Base containing S/R Flip, ChoCh, liquidity sweep, and Pinbar heuristics in prompt templates
- [ ] 4.2 Implement Planner & Reasoning Agent using Chain-of-Thought (CoT) prompts to synthesize spatial zones and ML probabilities, verifying reasoning trace consistency
- [ ] 4.3 Implement rule-based anomaly detector to reject setups with conflicting indicators and verify contradictory test cases are correctly blocked

## 5. Dynamic Risk Management & Decision Engine

- [ ] 5.1 Implement dynamic Stop-Loss (zone boundary + ATR buffer) and Take-Profit (opposing zone boundary) calculation, verifying math against test cases
- [ ] 5.2 Implement risk filter enforcing minimum Risk-to-Reward Ratio ($RR \ge 2.0$) and minimum win probability ($P \ge 0.65$), verifying filter rejection on sub-threshold setups
- [ ] 5.3 Implement position sizing calculator based on fixed fractional risk percentage and verify risk bounds do not exceed 2% per trade

## 6. Report Generation & System Integration

- [ ] 6.1 Implement Structured Report Interface producing JSON and Markdown trade briefs, verifying output against the defined schemas
- [ ] 6.2 Build end-to-end integration test executing the full pipeline from raw data to final report generation and verify end-to-end execution passes
- [ ] 6.3 Run historical backtesting evaluation comparing Hybrid TPA-Agent against traditional MA crossover and pure XGBoost strategies, verifying Win Rate, RR, and Max Drawdown metrics

## Workflow follow-up

- Validate all OpenSpec specs and change status.
- Sync delta specs to main specs upon completion.
- Archive the change after implementation and test verification are complete.

