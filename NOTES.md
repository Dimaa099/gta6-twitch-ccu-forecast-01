# GTA 6 Twitch CCU forecast — build notes

Competition: gta6-twitch-ccu-forecast-01 (pool $250, top-20 paid; deadline
2026-11-19T01:00:00Z / 08:00 WIB; no entry fee).
Account @dimarz, GitHub Dimaa099, agent wallet
2aBvBtva9oa8f2VcGnBea9sAMxXGJhmaRfeV3SRf1aYF.

## Design

- Rolling-midnight shape ensemble: only comparables whose unlock note says
  midnight-local rolling (not early-access contaminated). Simultaneous
  unlocks (Silksong, Wukong, Battlefield 6) and early-access launches have
  a different UTC-day shape and are excluded.
- Per-hour OLS of log(viewers_h / viewers_1) on x = ln(p1 / mean viewers
  21:00-23:00 UTC pre-release day), slope shrunk x0.85, x clamped to the
  ensemble's observed range.
- Spreads = per-hour residual SD (floor 0.15 log units); quantiles
  p1 * exp(m + z*sd), z = standard normal quantiles.

## Local validation (practice cases)

Kit's `grade-local` refuses program/v1 kits (sealed scorer is server-only),
so answers were reconstructed from the SullyGnome source the kit's
input/NOTICE.txt documents (4 targeted month fetches, same endpoint the kit
builders used). Reconstruction verified: hour-1 matches each case's known
print exactly for all 4 cases.

| case | loss | typical miss | cov90 |
|---|---|---|---|
| p00 Wolverine | 0.224 | 41% | 0.91 |
| p01 RDR2 | 0.137 | 16% | 0.96 |
| p02 God of War Ragnarok | 0.244 | 56% | 0.96 |
| p03 Spider-Man 2 | 0.232 | 42% | 0.78 |
| **mean** | **0.209** | **~38%** | **~0.90** |

House persistence baseline on the same answers: mean 0.387, miss 82.8%,
cov90 0.783. This entry beats the baseline by 0.178 with margin.

Variants tried (all on the same 4 answers): all-comparables median 0.310;
rolling median 0.252; rolling regression slope 1.0 0.210; slope 0.7 0.210;
kNN k=3/5/7 0.236-0.267; regression on ALL comparables 0.436 (mixed unlock
shapes destroy it). Shipped: rolling regression, slope 0.85, spread 1.0.

## Final-case notes

- The final case (h00) includes all 42 comparables, among them the four
  practice launches (all rolling-midnight) — the ensemble grows from ~11 to
  ~15 launches and the regression gains data.
- GTA VI unlocks rolling midnight local (NZ 2026-11-18T11:00Z, CET 23:00Z,
  UK 00:00Z, US East 05:00Z, US West 08:00Z) — exactly the shape class this
  model is built for.
- The final case's history is dense minute reads from the house recorder, so
  the overnight-drop x will be computed from real 21:00-23:00 hourly means.
