# Spec Delta

## Purpose

Generates standardized JSON and human-readable markdown reports containing trade execution parameters, evidence chains, and risk metrics.

## ADDED Requirements

### Requirement: Synthesize Structured Decision Report
The system SHALL aggregate validated trade setups, exact entry/stop-loss/take-profit prices, probability metrics, and reasoning logs into a structured output report.

#### Scenario: Approved trade setup report
- **WHEN** all analytical engines, ML thresholds, and risk filters approve a setup
- **THEN** the system outputs a complete JSON and formatted Markdown report labeled APPROVED with full entry specifications and reasoning summary

#### Scenario: No-trade wait report
- **WHEN** market conditions fail risk-reward or win-rate thresholds
- **THEN** the system outputs a status report labeled WAIT/NO_TRADE with detailed breakdown of failing criteria

