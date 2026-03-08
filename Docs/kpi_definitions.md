# KPI Definitions — Airline Operational Performance Analytics

## Fact Grain

- One row = one scheduled flight instance
- **Composite identity key:** `YEAR`, `MONTH`, `DAY`, `AIRLINE`, `FLIGHT_NUMBER`, `ORIGIN_AIRPORT`, `DESTINATION_AIRPORT`, `SCHEDULED_DEPARTURE`

## Operational Status Definitions

### Completed Flight

A flight is considered "completed" when it is not cancelled and not diverted:

- `CANCELLED = 0` AND `DIVERTED = 0`

### Cancelled Flight

- `CANCELLED = 1`
- **Note:** Some cancelled flights may have `DEPARTURE_TIME` and `DEPARTURE_DELAY` populated. Interpretation: cancellation indicates the flight did not complete the scheduled arrival, not necessarily that it never departed.

### Diverted Flight

- `DIVERTED = 1`
- Treated separately from cancellations

## Delay Definitions

### Primary Delay Metric

- Arrival delay minutes = `ARRIVAL_DELAY`

### On-Time Performance (OTP15)

- `is_on_time = 1` when `ARRIVAL_DELAY <= 15`
- **Eligibility:** Only completed flights (`CANCELLED = 0` AND `DIVERTED = 0`) are eligible for on-time evaluation

### Delayed (OTP15)

- `is_delayed = 1` when `ARRIVAL_DELAY > 15`
- Same eligibility as on-time

### Severe Delay (SD60)

- `is_severe_delay = 1` when `ARRIVAL_DELAY >= 60`
- Same eligibility as on-time

## Outlier Handling (No Capping)

- Delay values are not modified (no capping, no deletion)
- Extreme delays are flagged:
  - `is_extreme_arrival_delay = 1` when `ARRIVAL_DELAY < -60` OR `ARRIVAL_DELAY > 1440`
- KPIs include these rows by default; consumers may filter using the extreme flag

## Cancellation and Diversion KPIs

### Cancellation Rate

- `cancellation_rate` = (`CANCELLED = 1`) / all flights

### Diversion Rate

- `diversion_rate` = (`DIVERTED = 1`) / all flights

## Operational Disruption Flag

- `operational_disruption_flag = 1` when:
  - `CANCELLED = 1` OR `DIVERTED = 1` OR (eligible completed flight AND `ARRIVAL_DELAY >= 60`)

## Data Quality and Reference Integrity Notes

### Mixed Airport Identifiers

- Flight data originally contained mixed airport identifiers (IATA codes and DOT numeric IDs)
- A deterministic DOT → IATA mapping was created using canonical airport name + state matching
- 215 of 307 airports were successfully mapped; 92 remain unmatched
- Unmatched airports are documented in `unmatched_dot_airports.csv`
- Numeric identifier occurrences reduced from ~486k to ~141k in the fact table
- Remaining unmapped codes are retained with null descriptive attributes to preserve referential integrity
