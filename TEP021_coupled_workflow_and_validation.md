# TEP021: Coupled transient workflow and acceptance contract

## Summary

This TEP assembles the preceding state, transport, estimator, and plasma
proposals into a first-release workflow and defines its configuration and
acceptance policy.

## Status

**Discussion**

## Responsible

TBD

## Pull requests and Issues

None yet.

## Description

Time-dependent TARDIS needs an explicit coupling contract rather than a reuse
of the current single-epoch fixed-point loop. The first coupled scheme shall
be a first-order lagged operator split:

1. start with solved material state and radiation census at `t_k`;
2. build opacity and emissivity from `state[k]`;
3. propagate the census and new source packets through the physical interval;
4. normalize radiation and deposition tallies;
5. solve a trial material state at `t_(k+1)` using those tallies and endpoint
   local coefficients; and
6. validate and commit all state and ledgers atomically.

Only the material substep shall be backward Euler. The complete coupled step
shall be first order because transport coefficients are lagged. Accuracy shall
first be established by reducing the time step; predictor-corrector or Picard
coupling shall be added only if a defined target is not met.

Transient configuration shall live in a separate top-level `time_evolution`
section. It shall represent mode, time grid, source temperature history,
packet allocation, deposition source, and versioned acceptance profile.
Fixed first-release policies shall remain in the TEPs rather than becoming
undocumented configuration switches.

## Implementation

1. Add schema validation for `time_evolution` and atomic-rate coverage.
2. Integrate TEP016–019 in the roadmap order: prescribed-state finite-flight
   transport, prescribed-radiation matter evolution, quasi-static milestone,
   then finite-flight coupled evolution.
3. Add invariant checks before commit and clean failure at the prior solved
   state when a fixed-grid trial is rejected.
4. Add versioned acceptance profiles and calibration artifacts identifying
   equations, normalizations, seeds, packet budgets, environments, and
   thresholds.
5. Add transient analytic tests before each implementation behavior and
   maintain snapshot regression tests separately.

## Tests and validation

Acceptance shall distinguish:

- hard validity: finite, nonnegative state and no silent fallback or clamping;
- algebraic closure: scaled residuals over every applicable shell and equation;
- nuclei, charge, and energy conservation, each with its own scale;
- timestep accuracy over at least three successively refined grids;
- Monte Carlo adequacy using seeded ensembles and uncertainty-aware aggregate
  comparisons;
- bitwise restart equality in the fixed environment; and
- existing snapshot regression tolerances.

Thresholds shall be calibrated on analytic deterministic cases and seeded
benchmarks, versioned before release, and recorded in provenance. Runtime
overrides shall not silently loosen release or CI gates.

## Backward compatibility

The absence of `time_evolution` shall select the existing snapshot mode. The
new workflow shall not reinterpret `montecarlo.iterations`, existing scalar
luminosity inputs, or snapshot output timestamps as transient quantities.

## Alternatives

An immediate fully implicit radiation–matter solve would be more tightly
coupled but would require a larger nonlinear system and substantially more
complex rollback. A first-order operator split provides a smaller validated
increment. Automatic step reduction is desirable but may be deferred until the
fixed-grid failure and diagnostic contract is working.

