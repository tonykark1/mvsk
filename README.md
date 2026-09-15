# MVSK out-of-sample portfolio optimization stress test

This repository studies whether higher-moment portfolio optimization (mean-variance-skewness-kurtosis, **MVSK**) improves out-of-sample portfolio performance once the experiment is made harder to game: persistent random universes, point-in-time index membership, transaction costs, multiple portfolio sizes/horizons, placebo tests, bootstrap inference, and explicit numerical diagnostics.

## Current scientific status

> **v17 completed successfully, but its empirical conclusions are not treated as final.**
>
> A post-run solver audit found that the intended eight-start SOLNP protection behaved largely like a one-reliable-start procedure. The seven random starts were typically boundary-heavy and often nonconvergent. A repaired 20-episode validation found materially better MVSK optima in 4/20 episodes, including 3 severe improvements. A corrected v18 rerun is therefore required before the strategy results are interpreted as definitive.

This is exactly the kind of failure the audit layer is meant to catch. The v17 outputs remain useful for debugging and research design, but they should **not** be cited as final evidence for or against MVSK.

## Research question

The main question is not simply "does MVSK work?" It is:

> **How robust is the apparent benefit of higher-moment portfolio optimization to universe selection, estimator instability, transaction costs, out-of-sample design, and numerical optimization?**

The project is designed as a stress test rather than as a new optimizer proposal.

## v17 experiment design

The frozen v17 run uses:

- three train/hold designs: **3y/3m, 5y/6m, 5y/12m**;
- portfolio sizes **N = 20, 50, 100**;
- **U = 100 persistent universe tracks** per N;
- long-only, fully invested portfolios with a **15% single-name cap**;
- the locked `MVSK_EMP_XI2` higher-moment specification against MV and related controls;
- realized turnover/cost accounting;
- **19 placebo** perturbations;
- **5,000 bootstrap draws** for the final inference layer;
- exact serial-vs-parallel SOLNP verification;
- U-convergence diagnostics at multiple prefixes of the 100 universe tracks.

## Free-data point-in-time approximation

The completed free-data run reconstructed historical S&P 500 membership using exact effective dates and cached Yahoo histories. Exact preprocessing contained approximately:

- **735 historical tickers** requested from cache;
- **595 securities** surviving preprocessing;
- **2,458,000 return rows**;
- **20,940 membership snapshots**;
- **46 exact cutoff dates**.

This is intentionally labelled a **free-data approximation, not CRSP-quality data**. Important caveats remain: ticker is not a permanent security identifier; some historical securities are missing from Yahoo; there is no authoritative CRSP-style `DLRET`; historical sector and market-cap information is incomplete; ADV60 is used as a practical liquidity/size proxy; and historical data availability can create residual survivorship/data-availability bias.

A later licensed-data replication would strengthen the academic version, but it is not required to debug or validate the numerical research design.

## Solver audit: why v17 is not final

The quick validation reproduced the saved v17 optimization state exactly, then replaced the original boundary-heavy random starts with strictly interior feasible starts.

### Original start geometry

Median across sampled random starts:

- **39 exact-zero weights**;
- **2 weights at the 15% cap**;
- effective holdings: **8.26**.

### Repaired interior starts

Median across repaired starts:

- **0 exact-zero weights**;
- **0 weights at the cap**;
- effective holdings: **35.88**;
- **20/20 starts converged** in the median episode.

### 20-episode triage result

- saved-vs-reconstructed utility error: **0**;
- episodes with material objective improvement: **4/20**;
- episodes with severe improvement: **3/20**;
- episodes with materially different weights: **4/20**;
- maximum utility improvement: **0.002889**;
- convex MV QP controls passed **20/20**;
- local perturbation search found no nearby improvement around the repaired optima.

The audit verdict is therefore **FAIL** for the old multistart protocol: the repaired starts can locate meaningfully better nonconvex MVSK solutions. See [`results/solver_quick_validation/SUMMARY.txt`](results/solver_quick_validation/SUMMARY.txt) and [`docs/SOLVER_VALIDATION.md`](docs/SOLVER_VALIDATION.md).

## What happens next

v18 will change the numerical protocol before another full U=100 run:

1. replace the boundary-heavy random-start generator with dense, strictly interior feasible starts;
2. permit only `convergence == 0` solutions to compete for the selected optimum;
3. save per-start convergence and feasibility diagnostics honestly;
4. validate the corrected optimizer on a small pilot first;
5. rerun the frozen U=100 experiment only after the pilot passes.

The economic/statistical specification should otherwise remain fixed so that any change in results can be attributed to the numerical correction rather than to researcher discretion.

## Reproducibility principle

A finite feasible optimizer output is not treated as proof of optimality. For nonconvex MVSK problems, the project now requires convergence-aware multistart diagnostics and independent numerical controls before economic conclusions are accepted.

## Repository layout

```text
docs/
  SOLVER_VALIDATION.md

results/solver_quick_validation/
  SUMMARY.txt
  episode_solver_validation.csv
```

Large raw data caches and full backtest checkpoint trees are intentionally not committed.
