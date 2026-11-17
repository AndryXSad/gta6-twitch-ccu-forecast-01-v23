# DESIGN.md — gta6-twitch-ccu-forecast-01 v23

## The bar
- v2 (verified, direct-graded): practice **0.128297** (p00 0.1215 / p01
  0.1296 / p02 0.0907 / p03 0.1713), LOO-large 0.2113, LOO-mega 0.2137.
- v22 (the previous best, direct-graded): practice **0.127258** (p00
  0.1223 / p01 0.1311 / p02 0.0900 / p03 0.1656), LOO-large 0.210621,
  LOO-mega 0.212838 — the first version in 22 rounds to beat v2 on BOTH
  practice and LOO.
- House baseline: 0.387077. Live bar at 17 Nov ~17:27 WIB: #1 0.0975,
  top-10 ≈ 0.115, top-20 ≈ 0.122, ~3,127 entries. Prizes pay top 20.
- Owner's rule: keep improving until a candidate hits practice ≤ 0.100,
  then submit it. This version does not reach 0.100 — it refines v22's
  winning plane and beats v22 on all three numbers.

## What v23 is
A 3-way weighted-mean blend (45% Y5 / 35% Z0 / 20% clone) with
**per-band asymmetric spread scales**, a coordinate refinement of v22's
winning triple:

| band | hours | lower (q05/q25) | upper (q75/q95) | v22's triple |
| ---- | ----- | --------------- | --------------- | ------------ |
| early | 2–6 | 0.90 | 1.10 | (0.85, 1.05) |
| mid | 7–14 | 1.10 | 0.80 | (1.05, 0.85) |
| late | 15–24 | 0.95 | 0.85 | (0.95, 0.85) |

Direct-graded practice **0.126869** (p00 0.1227 / p01 0.1323 / p02 0.0896 /
p03 0.1628); LOO-large **0.210446**, LOO-mega **0.212649**. A second
cell from the same one-band-at-a-time refinement — early (0.85, 1.05),
mid (1.10, 0.80), late (0.95, 0.85) — direct-graded **0.126933** with
LOO 0.210584 / 0.212826, confirming the refinement is not a one-cell
fluke: both graded cells beat v22 on all three numbers.

## Why this shape
Twenty-two rounds measured the same frontier: every global spread scale
trades practice against LOO in the same ratio (v14), because the two
error regimes want opposite spread shapes — practice's worst misses are
early-hour troughs below the median (punished less by a narrow lower
side), while LOO launches' worst misses are mid-day drift above the
median (v16: medians biased HIGH on LOO-large, mean log-residual ≈
−0.05, so the lower side needs width). v22's per-band split was the
first construction to move the corner. This round probed around it:
the one-band refinement grid (25 cells) has 4 cells passing v22's
gates, and the packaged cell widens the early upper side and the mid
lower side together — the two directions the passing cells agree on.

## How the packaged cell was found (honest note)
The packaged triple was NOT in the one-band grid: it came out of a
variant-building slip while writing the confirmation program for the
second passing cell (the mid band was left at the first cell's (1.10,
0.80) instead of the second cell's (1.05, 0.85)). Its direct grade
(0.126869) came back better than the intended cell's harness number,
so the triple was checked in the harness — 0.126869 / 0.210446 /
0.212649, matching the direct grade exactly — and then adopted. It is
a genuine two-band refinement of the winner, verified the same way as
every other candidate (harness + direct grade agree to 6 decimals),
but it was not selected by a systematic grid: treat the LOO margin
(−0.000175) as selection-tinged, as with all LOO-chosen cells.

## The honest caveat (read this before trusting the number)
The band parameters were chosen on LOO-large, so LOO-large is **not a
clean holdout** for the chosen cell — its margin is optimistically
biased by selection. The practice margin (−0.000389 vs v22) is the
cleaner evidence, and it was confirmed by direct grading of the actual
program (0.126869, identical to the harness to 6 decimals). The
refinement neighbourhood holds together — 4 of 25 one-band cells pass
v22's gates, and both direct-graded cells beat v22 on all three
numbers — but the final is a new launch in a new regime (lnV1 ≈ 14+,
beyond the library max 13.68), and practice is only 4 launches. Treat
this as the best verified fallback, not a guarantee. The median is
still the bottleneck: best median-only loss ever is 0.1975 (Y5); this
blend's median-only loss is 0.1989 practice / 0.3253 LOO-large.

## Members (all in log space)
- **Y5**: v4-family member — tier EA filter, per-hour OLS on
  [lnV1, g12], shape views (console-only + all-launch, windowed,
  population sd), mix 0.5/0.2/0.3, sqrt-variance blend floored at
  sqrt(0.02). Best median-only loss ever seen (0.1975).
- **Z0**: same construction, naive 'early access' (space) filter.
- **Clone**: v9 black-box reconstruction of v2's spread rule — whole
  library shape pools, ≥20-hour fits, 3-feature shape regressions,
  sample sd everywhere, OLS pool counted after the hour filter,
  weighted-mean blend with spread = 0.81 × weighted mean of spreads,
  mix 0.6/0.2/0.2.

Features: lnV1 = log hour-1 viewers; lnPREM = log MEAN viewers in the
7 days before release day; g12 = lnV1 − log viewers 12h before release
(nearest point within 3h, library-median fallback, upper median).
Size fed to the regressions is clamped to the library's lnV1 range +
0.5 headroom. Output rows are explicitly forced non-decreasing.

## What was tried and dropped in v23
- Per-band asymmetric scales on the cross-pairings (idea 1): 729 cells
  each for Y5 median + clone spread and ensemble median + Y5 spread —
  **0 cells pass v22's gates** in either pairing (closest: ensemble
  median + Y5 spread, triple ((0.85,1.05),(1.05,0.85),(0.85,0.85)),
  practice 0.127558 — fails only practice by +0.000300; Y5 median +
  clone spread closest joint miss +0.000621). 7 and 16 cells pass the
  old v2 gates, which no longer count.
- 3-way blend weights x v22's winning triple (idea 3): (0.4, 0.3, 0.3)
  practice 0.126971 but LOO 0.211332; (0.5, 0.25, 0.25) practice
  0.127103 but LOO 0.211135 — weights are still almost a non-lever
  (v20's finding holds at the refined triple too).
- Earlier rounds (v3–v22) had already closed: new median families,
  global/per-band symmetric scales, per-hour LOO-spread, empirical
  quantile offsets, bias corrections, channels/roughness/unlock views,
  streams/categories hybrids, robust/quantile/local/GP/quadratic/
  peak-aligned medians, pooled libraries, per-band feature sets,
  per-hour windows, fitted blend weights, inverse-roughness shapes,
  median-of-views members.

## Verification
- Harness calibration (fresh harness, exact): house 0.387077; v2's
  actual main.py reproduces 0.128297 exactly; clone 0.129250 (LOO
  0.2198/0.2214); Y5 0.130685 (LOO 0.2095/0.2118); ensemble 0.127632
  (LOO 0.2147/0.2171); half-blend 0.126261 (LOO 0.2219/0.2238);
  3-way x0.92 0.128311 (LOO 0.211954/0.214060). The harness's
  recomputed member rows match v22's cached rows with max |diff|
  0.00e+00.
- This program direct-graded on the 4 practice cases with the exact
  RULES scorer: **0.126869** (harness 0.126869 — exact, no gap: the
  program uses the same upper-median fallback as the harness).
  Runtime ~0.2 s/case, deterministic (2 runs byte-identical), output
  23 rows × 5 finite non-decreasing quantiles in (0, 1e9].
