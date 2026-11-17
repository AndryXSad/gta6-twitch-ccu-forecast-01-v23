# GTA VI launch-day Twitch CCU forecaster — v23

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/AndryXSad/gta6-twitch-ccu-forecast-01-v23/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/AndryXSad/gta6-twitch-ccu-forecast-01-v23/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

A make.fun `gta6-twitch-ccu-forecast-01` entry. It forecasts, for each of
hours 2–24 of Grand Theft Auto VI's release day (2026-11-19 UTC), five
quantiles (5%, 25%, 50%, 75%, 95%) of the concurrent Twitch viewers in the
"Grand Theft Auto VI" category, from the first hour's print and the
launch-day curves of past big game launches shipped inside each case.

## How it works

`main.py` (Python 3.13, standard library only, no network) blends three
member predictors in log space:

1. **Y5** — tier-filtered per-hour OLS on `[lnV1, g12]` blended
   (0.5/0.2/0.3) with console-only and all-launch one-factor shape views.
   `g12` is the "unlock momentum" feature: lnV1 minus log viewers 12 hours
   before release day — the rolling-midnight wave already underway when the
   release day starts.
2. **Z0** — the same construction with the naive early-access filter
   (launches whose notes contain "early access" are excluded).
3. **Clone** — whole-library shape views, sample-sd spreads, and a
   weighted-mean blend (0.6 OLS / 0.2 console-shape / 0.2 all-shape) whose
   spread is 0.81× the weighted mean of the view spreads.

Final blend: 45% Y5 + 35% Z0 + 20% clone, for both the log-median and the
log-spread. Then **per-band asymmetric spread scales**: early hours 2–6 use
(lower 0.90, upper 1.10), mid hours 7–14 use (1.10, 0.80), late hours 15–24
use (0.95, 0.85). The lower side is wider mid/late because the medians run
high there (mean log-residual about −0.05 on large launches); the early
upper side is wider because the opening ramp's upper tail matters most
then. Quantiles are lognormal: q = exp(mu + z · sig_side), with the
output rows explicitly forced non-decreasing.

An extrapolation guard clamps the size fed to the regressions to the
library's observed lnV1 range plus half a log unit of headroom (the level
anchor stays the true hour-1 print).

Full reasoning, the numbers behind every choice, and what was tried and
dropped: [DESIGN.md](DESIGN.md).

## Local results (practice cases, scored with the exact pinball loss from RULES.txt)

| case | launch | loss |
| ---- | ------ | ---- |
| p00 | Marvel's Wolverine (2026-09-15) | 0.123 |
| p01 | Red Dead Redemption 2 (2018-10-26) | 0.132 |
| p02 | God of War Ragnarök (2022-11-09) | 0.090 |
| p03 | Marvel's Spider-Man 2 (2023-10-20) | 0.163 |
| **overall** | | **0.127** |

For reference: the house baseline scores 0.387 on the same cases, v1 scored
0.154 (judge loss 0.154426, rank #181), v2 scores 0.128297, and v22 — the
previous best — scores 0.127258. Leave-one-out over the comparable
library: large launches (lnV1 ≥ 11) 0.210, mega launches (lnV1 ≥ 11.5)
0.213 — both below v22's 0.211 / 0.213.

## Usage

```sh
MAKEFUN_INPUT_DIR=/input MAKEFUN_OUTPUT_DIR=/output python3 main.py
```

Reads every `*.in` in the input directory, writes `<id>.out` (23 rows,
hours 2–24, five quantiles each) to the output directory. Deterministic,
~0.2 s per case.
