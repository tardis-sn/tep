# TEP017: Time-dependent state and reproducible evolution

## Summary

This TEP defines the physical time grid, solved-state ownership, homologous
matter evolution, checkpointing, and reproducibility contract required by
time-dependent TARDIS.

## Status

**Discussion**

## Responsible

TBD

## Pull requests and Issues

None yet.

## Description

Current TARDIS constructs one snapshot at `supernova.time_explosion`.
Monte Carlo iterations are convergence iterations, not physical time steps,
and the existing HDF products are not complete restart checkpoints. A
transient calculation therefore needs one explicit owner for physical time and
one complete state that can be committed or discarded.

The first release shall support a monotonic physical grid with boundaries
`t_k` and `t_(k+1)`. Transport intervals shall be half-open,
`[t_k, t_(k+1))`; a packet reaching the end boundary shall enter the next
census. State, composition, geometry, census, checkpoints, and ledgers shall
be associated with time-step boundaries, while coefficients derived from
`state[k]` shall be held over the following interval.

Radiation shall use the inertial centre-of-ejecta lab frame for packet time,
position, direction, frequency, and energy. Local comoving quantities shall be
calculated at events using the exact Lorentz transformation. The Newtonian
homologous matter model shall remain separate from observer arrival time,
which shall be left to postprocessing.

Exactly one solved state shall own the current physical time. A trial state
shall become solved only after solver, conservation, and bookkeeping checks
pass. Matter, census, source progress, RNG progress, outputs, and energy
ledgers shall commit together. A failed trial shall leave the previous solved
state unchanged.

For homologous shells, the implementation shall preserve velocity boundaries,
reference density, shell mass fractions, and mass-normalized ion abundances.
Radii, volumes, densities, ion fractions, and number densities shall be
derived. Decay histories shall be precomputed for the configured grid, with
all reachable daughter elements allocated and validated before evolution.

Checkpoints shall be readable at solved boundaries and shall contain complete
state, packet, RNG, source, ledger, provenance, and diagnostic information.
Continuation in the same supported environment and parallel layout shall be
bitwise identical to uninterrupted execution. Different layouts shall agree
statistically rather than bitwise.

## Implementation

1. Add a validated physical time-grid object and explicit time associations for
   states, coefficients, estimators, outputs, and diagnostics.
2. Separate snapshot Monte Carlo iteration indices from physical-step and
   coupling-iteration indices.
3. Introduce solved and trial state containers with atomic commit and complete
   rollback.
4. Advance homologous geometry, density, isotope inventories, and decay
   sources without changing shell mass or applying dilution twice.
5. Define a versioned checkpoint schema and restore path, including packet
   identities and consumed RNG state.
6. Add deterministic ordering and reduction rules needed by the restart
   contract.

Steps 1–3 precede all transient transport. Step 4 may be developed alongside
   plasma evolution. Step 5 is required before claiming restart support.

## Tests and validation

- Reject invalid, duplicate, or non-monotonic boundaries before model creation.
- Verify analytic homologous radius, volume, density, and invariant shell mass.
- Verify decay parent-loss and daughter-birth conservation, including a
  daughter first appearing from zero abundance.
- Force a failed trial and confirm that no solved arrays, counters, source
  energy, or composition change.
- Interrupt at a solved boundary and require bitwise-identical continuation in
  a fixed environment; compare alternate layouts statistically.

## Backward compatibility

Configurations without `time_evolution` shall continue to run the existing
single-epoch snapshot workflow. Existing Monte Carlo iterations, normalization,
outputs, and snapshot regression tolerances shall retain their current meaning.

## Alternatives

The project could mutate the existing `SimulationState` in place or use Monte
Carlo iterations as time steps. Both approaches conflate physical time with
convergence state and make rollback and restart incomplete. A full
hydrodynamic history is also deferred: it would require mesh identity or
remapping and time-dependent velocity and resonance support.

