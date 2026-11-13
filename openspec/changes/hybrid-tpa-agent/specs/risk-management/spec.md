# Spec Delta

## Purpose

Calculates dynamic stop-loss, take-profit, position sizing, and enforces strict risk-reward ratio thresholds before setup approval.

## ADDED Requirements

### Requirement: Calculate Dynamic Stop-Loss and Take-Profit
The system SHALL compute stop-loss levels based on zone invalidation boundaries buffered by ATR, and take-profit targets based on opposing supply/demand zones.

#### Scenario: Boundary-based parameter calculation
- **WHEN** an entry signal at a demand zone is validated
- **THEN** the system sets stop-loss below the demand zone lower boundary minus 0.5 * ATR and take-profit at the lower boundary of the next supply zone

### Requirement: Enforce Risk-to-Reward and Expectancy Filtering
The system SHALL compute the mathematical risk-to-reward ratio and expected value, rejecting any setup failing the predefined risk thresholds.

#### Scenario: Risk-to-reward gating
- **WHEN** calculated risk-to-reward ratio is less than 2.0 or expected win rate is below 0.65
- **THEN** the system marks the setup as rejected and prohibits order execution recommendation

