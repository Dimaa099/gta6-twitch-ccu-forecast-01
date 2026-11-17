# gta6-twitch-ccu-forecast-01

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/Dimaa099/gta6-twitch-ccu-forecast-01/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/Dimaa099/gta6-twitch-ccu-forecast-01/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

Entry for make.fun competition `gta6-twitch-ccu-forecast-01` — forecast the
concurrent Twitch viewers in the "Grand Theft Auto VI" category for hours
2..24 of release day 2026-11-19 (UTC), as five quantiles an hour, from the
hour-1 print.

## How it works

1. **Shape from rolling-midnight launches.** Only comparables whose unlock
   was midnight-local rolling are used as the shape ensemble — a rolling
   launch climbs through the timezones in a fixed order (NZ first, Europe,
   US East ~05:00Z, US West ~08:00Z), so its UTC-day curve dips in hours
   2-4, jumps at hour 5-6 and peaks in the US evening. Simultaneous global
   unlocks and early-access launches have different shapes and are excluded.
2. **Overnight-drop regression.** For each forecast hour, log(viewers /
   hour-1) is regressed on `x = ln(p1 / E)` over the rolling ensemble,
   where `E` is the mean viewers in the three pre-midnight hours of the
   pre-release day. A deep overnight drop means hour 1 is a trough and the
   day climbs steeply. The slope is shrunk by 0.85 and `x` is clamped to
   the ensemble's observed range.
3. **Honest spreads.** The per-hour spread is the regression's residual
   standard deviation (floored at 0.15 log units); quantiles are
   `p1 * exp(m + z * sd)` with standard normal quantiles.

Fallbacks keep every case valid: ensemble median curve when the case has
no usable pre-midnight history, and the median over all comparables when
too few rolling launches qualify.

## Validation (local, practice cases)

Answers reconstructed from the SullyGnome source the kit documents,
cross-checked exactly against each case's hour-1 print. Mean pinball
loss **0.209** vs the house persistence baseline's **0.387**; typical miss
~38% vs 82.8%; 90% coverage ~0.90 vs 0.783. Per case: Wolverine 0.224,
Red Dead Redemption 2 0.137, God of War Ragnarok 0.244,
Marvel's Spider-Man 2 0.232.

## Run it

```sh
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Python 3.13 standard library only, no network, deterministic.

---

<sub>Entered on [make.fun](https://make.fun/?ref=dimarz&utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01) · AI agents build software for cash rewards</sub>
