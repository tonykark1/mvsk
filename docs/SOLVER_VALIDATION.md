# Solver validation note

## Finding

The completed v17 backtest was numerically reproducible, but its intended eight-start SOLNP safeguard was not functioning as intended.

The seven random starts were usually very sparse and boundary-heavy. In the quick audit, the original random-start geometry had a median of 39 exact-zero weights and 2 weights at the 15% cap. This can be difficult for SOLNP because many inequality constraints are active immediately.

The quick validator reconstructed 20 saved episodes exactly and then re-solved the locked `MVSK_EMP_XI2` objective with 20 dense, strictly interior feasible starts. Only solutions with `convergence == 0` were allowed to compete.

## Validation controls

The audit included:

- exact reconstruction of the saved episode objective;
- 20 representative episodes stratified over all three horizons and N=20/50/100;
- 20 strictly interior starts per episode;
- strict convergence filtering;
- feasibility checks;
- numerical projected-gradient / local perturbation checks;
- a convex mean-variance control solved with `quadprog`.

The maximum saved-vs-reconstructed MVSK utility error was exactly 0. The convex MV QP control succeeded in all 20 episodes and did not flag an implementation problem.

## Result

The repaired multistart procedure found:

- 4/20 episodes with a material objective improvement;
- 3/20 with a severe improvement;
- 4/20 with materially different portfolio weights;
- maximum MVSK utility improvement of 0.002889.

The three severe improvements occurred in sampled `5y_12m` episodes. The sampled primary `5y_6m, N=50` cases were more stable, but one case still produced materially different weights with a smaller objective gain.

The quick-audit verdict is **FAIL** for the v17 optimizer protocol. This does **not** prove that the economic conclusion reverses. It proves that v17 cannot be treated as a definitive test because the numerical search was weaker than intended.

## Scientific consequence

The correct response is not to delete the result or selectively remove failed episodes. The numerical protocol must be corrected and frozen, validated on a pilot, and then the full experiment rerun under the same economic/statistical specification.

## Planned v18 correction

- strictly interior multistart generation;
- only `convergence == 0` solutions eligible for selection;
- explicit per-start convergence/feasibility reporting;
- pilot validation before a new U=100 run;
- no discretionary changes to the locked economic model or inference design during the solver correction.

The purpose of v18 is to isolate the effect of the numerical correction.
