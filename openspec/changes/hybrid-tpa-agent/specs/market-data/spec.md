# Spec Delta

## Purpose

Provides validated OHLCV candlestick data and volume profile calculations across multiple timeframes for downstream zone detection and machine learning feature extraction.

## ADDED Requirements

### Requirement: Ingest Historical and Real-time OHLCV Data
The system SHALL ingest historical and real-time OHLCV market data for specified trading pairs and timeframes, validating timestamp continuity and data completeness.

#### Scenario: Successful data fetch
- **WHEN** a valid symbol and timeframe are requested
- **THEN** the system returns a contiguous pandas DataFrame containing timestamp, open, high, low, close, and volume columns without null values

#### Scenario: Missing or corrupt data handling
- **WHEN** incoming candlestick data contains missing bars or corrupted volume numbers
- **THEN** the system logs an error, rejects the batch, and attempts recovery or raises a validation exception

### Requirement: Calculate Volume Profile
The system SHALL compute horizontal volume distribution across price bins over a user-defined lookback window to identify High Volume Nodes (HVN) and Low Volume Nodes (LVN).

#### Scenario: Volume profile computation
- **WHEN** OHLCV data is provided with a specified number of price bins
- **THEN** the system outputs price bins mapped to total traded volume and marks the Point of Control (POC)

