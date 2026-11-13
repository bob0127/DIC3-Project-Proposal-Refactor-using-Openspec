# Spec Delta

## Purpose

Extracts spatial, momentum, and volume features and computes probabilistic win rates for zone breakouts and retest bounces using trained XGBoost classification models.

## ADDED Requirements

### Requirement: Technical and Spatial Feature Extraction
The system SHALL compute normalized technical indicators, volume momentum metrics, and spatial distance vectors from the current price to the nearest active demand and supply zones.

#### Scenario: Complete feature vector generation
- **WHEN** raw OHLCV and detected zones are passed to the feature extractor
- **THEN** the system returns a tabular feature vector including RSI, ATR, Volume Z-Score, and normalized distance to nearest zones

### Requirement: Predict Breakout and Retest Probabilities
The system SHALL infer the probability of a successful zone retest bounce versus breakout using an optimized XGBoost classifier.

#### Scenario: Inference execution
- **WHEN** a feature vector is input to the inference pipeline
- **THEN** the model outputs calibrated probability scores between 0.0 and 1.0 for bounce and breakout outcomes

#### Scenario: Model prediction thresholding
- **WHEN** the predicted bounce probability is calculated
- **THEN** the system flags whether the probability meets or exceeds the required threshold of 0.65

