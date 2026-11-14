# Polymarket FAA outage attribution forecaster

Forecasts the Yes price of every market in Polymarket's FAA outage
attribution family, minute by minute, 4 hours ahead: five quantiles
(5%, 25%, 50%, 75%, 95%) per market per minute.

## Method

For each market, from that case's own up-to-72h of 1-minute prices:

1. **Empirical return quantiles.** At a grid of horizons (1–240 min),
   the distribution of h-minute returns over the market's history,
   recency-weighted with a 90-minute half-life, centred on its median
   and interpolated in log-h. The short half-life adapts fast after
   jumps, which dominate this market's risk.
2. **Small median drift.** Momentum from the last 5 minutes
   (0.25 × r5) plus mean reversion from the last hour (−0.10 × r60),
   ramped linearly over the horizon. Both slopes were measured on the
   practice cases and kept small on purpose.
3. **Gated cross-market calibration.** When the family's checkpoint
   prices sum near 1 (a complete mutually-exclusive set, like the
   practice's NVIDIA CEO candidates), each median is shifted toward a
   total of 1, shared in proportion to price. The gate is essential:
   the final's four FAA attribution markets are not mutually exclusive
   (Iran ⊂ "foreign actor", and "no attribution" is a live outcome) —
   their live prices sum to ~0.10, so forcing a sum of 1 would inflate
   every median ~10x. The gate is evaluated per case from that case's
   own checkpoint prices.
4. **Short-history fallback.** A Gaussian spread from the median
   absolute 1-minute move, floored at 0.0004 per √minute.

Deterministic, standard library only, no network. ~0.5 s per case
(limit 8 s).

## Local backtest (practice cases)

Scored with the pinball loss against the actual post-checkpoint prices
fetched from Polymarket's public price history:

| case | this entry | house baseline |
|------|-----------|----------------|
| p00  | 0.446439  | 0.536138      |
| p01  | 0.307473  | 0.380352      |
| p02  | 0.346621  | 0.408408      |
| p03  | 0.312896  | 0.368599      |
| p04  | 0.194099  | 0.298794      |
| p05  | 0.119903  | 0.238232      |
| **overall** | **0.278459** | **0.363589 (−23.4%)** |

Wins 6/6 practice cases. The local scorer reproduces the house-quoted
baseline (0.363589) exactly. Note the practice is cut from the NVIDIA
CEO event, not the FAA markets, so it validates mechanics and
calibration method only — the gate above is what carries the method to
the final's very different price regime.

## Run

```sh
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```
