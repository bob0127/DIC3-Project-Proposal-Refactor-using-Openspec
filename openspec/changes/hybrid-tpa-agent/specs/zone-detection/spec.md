# Spec Delta

## Purpose

Detects swing highs, swing lows, horizontal support and resistance levels, and volume-weighted demand and supply zones using 100% deterministic algorithms.

## ADDED Requirements

### Requirement: Deterministic Swing Point Identification
The system SHALL identify local Swing Highs and Swing Lows using rolling window extrema identification without lookahead bias.

#### Scenario: Swing high and low extraction
- **WHEN** OHLCV series is processed with a specified left and right bar window
- **THEN** all pivot high and pivot low timestamps and prices are extracted deterministically

### Requirement: Identify Supply and Demand Zones
The system SHALL cluster pivot levels and detect strong directional imbalance candles to construct discrete upper and lower price boundaries for Supply and Demand Zones.

#### Scenario: Valid demand zone creation
- **WHEN** a strong bullish expansion candle originates from a consolidation base
- **THEN** the system creates a demand zone bounded by the candle low and body open, assigning zone strength based on volume and retest counts

#### Scenario: S/R flip detection
- **WHEN** price decisively closes below a previously confirmed demand zone with high volume
- **THEN** the system updates the zone role to supply resistance according to S/R flip rules

