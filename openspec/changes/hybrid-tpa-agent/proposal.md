# Proposal: Hybrid Technical & Price Action Agent (Hybrid TPA-Agent)

## Why

Intraday and swing traders in volatile financial markets (e.g., cryptocurrency and equities) suffer from severe cognitive overload, indicator lag, and emotional biases (fear, greed, FOMO). While Price Action and Supply & Demand (S&D) zone analysis provide powerful spatial market frameworks, manual identification is subjective and cognitively demanding under fast-paced market conditions. Existing quantitative solutions rely on lagging indicators or black-box ML without spatial perception, while pure LLM financial agents suffer from severe numerical hallucinations and lack deterministic boundary precision. 

The Hybrid TPA-Agent solves this by establishing a decoupled hybrid paradigm: 100% deterministic spatial zone detection, XGBoost probabilistic win-rate prediction, and LLM-driven Chain-of-Thought (CoT) contextual reasoning and dynamic risk management. This provides traders with objective, mathematically rigorous, and auditable high-probability trading setups.

## What Changes

This change introduces the full foundational architecture and pipeline for the Hybrid TPA-Agent:
- **ADDED**: Deterministic Price Action & S&D Zone Detection Engine (`zone-detection`) to algorithmically identify swing highs/lows, S/R lines, and volume-weighted demand and supply zones.
- **ADDED**: Real-time and historical market data acquisition with volume profile calculation (`market-data`).
- **ADDED**: Machine Learning Win-Rate Classifier (`win-rate-prediction`) using XGBoost to predict breakout vs. bounce probabilities based on engineered spatial and momentum features.
- **ADDED**: LLM Planner & Reasoning Agent (`reasoning-agent`) combining price action domain knowledge (S/R flips, liquidity sweeps) and multi-step Chain-of-Thought reasoning.
- **ADDED**: Decision & Risk Management Engine (`risk-management`) dynamically calculating precise stop-loss, take-profit, position size, and asymmetric Risk-to-Reward ratio ($RR \ge 2.0$, Win Rate $\ge 65\%$).
- **ADDED**: Structured Trading Report Interface (`report-generation`) outputting actionable, auditable execution setups with full reasoning evidence chains.

## Capabilities

### New Capabilities
- `market-data`: Ingestion, validation, and preprocessing of real-time and historical OHLCV candlestick data and volume profile calculation.
- `zone-detection`: Deterministic detection of swing pivots, horizontal support/resistance levels, and volume-weighted supply and demand zones without LLM hallucination.
- `win-rate-prediction`: Feature extraction and XGBoost classification for calculating probabilistic win rates of zone retests and breakouts.
- `reasoning-agent`: Multi-step LLM reasoning agent utilizing Price Action domain knowledge rules to evaluate market structure context.
- `risk-management`: Calculation of dynamic stop-loss, take-profit, position sizing, and filtering based on expected value and risk-to-reward ratios.
- `report-generation`: Synthesis of numerical parameters, confidence scores, and reasoning traces into standardized JSON and human-readable markdown reports.

### Modified Capabilities
None. (This is an initial greenfield implementation).

## Impact

- **Core Dependencies**: Python 3.10+, pandas, numpy, scikit-learn, xgboost, pydantic, langchain/openai/anthropic API SDKs.
- **Architecture**: Enforces strict decoupling between deterministic numerical computation (Python) and semantic reasoning (LLM), preventing numerical hallucinations.
- **APIs**: Standardized JSON data contracts between data acquisition, zone detection, ML inference, and LLM reasoning modules.

