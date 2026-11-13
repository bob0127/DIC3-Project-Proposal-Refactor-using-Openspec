# Spec Delta

## Purpose

Executes Chain-of-Thought context evaluation and market structure interpretation using structured domain knowledge rules to validate trading setups.

## ADDED Requirements

### Requirement: Evaluate Market Context with Domain Knowledge
The system SHALL ingest deterministic zone coordinates, ML probability scores, and price action domain rules into an LLM reasoning agent to perform structured Chain-of-Thought validation.

#### Scenario: CoT validation of setup
- **WHEN** the current market state and high ML probability setup are passed to the agent
- **THEN** the agent outputs step-by-step reasoning evaluating candle patterns, multi-timeframe alignment, and liquidity sweep evidence

#### Scenario: Conflicting signals rejection
- **WHEN** ML indicates high bounce probability but candle close shows an aggressive breakout or macro news anomaly
- **THEN** the reasoning agent detects the conflict and rejects the trade setup with an explicit rationale

