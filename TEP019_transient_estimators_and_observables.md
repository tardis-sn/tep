# TEP019: Transient estimators, energy accounting, and observables

## Summary

This TEP defines physical-time estimator normalization, coupled energy ledgers,
and escape-time observables for finite-flight TARDIS.

## Status

**Discussion**

## Responsible

TBD

## Pull requests and Issues

None yet.

## Description

Current estimators use snapshot volumes and artificial steady durations, while
current spectra describe terminal packets rather than a physical escape-time
series. A transient workflow shall retain additive, typed time-step tallies
before normalization and shall distinguish radiation-rate estimates from
conservation-ledger energies.

Every tally shall identify units, frame, time-step identity, normalization,
and uncertainty information where applicable. For a homologous shell
`V_i(t)=C_i t^3`, each path or resonance contribution shall apply the exact
`1/V_i(t)` factor inside the integral. Splitting a trajectory at a cell,
source, or time boundary shall not change its tally. The plasma interface shall
consume physical radiative rates rather than only fitted `T_rad` and `W`.

Radiation terms in the energy ledger shall use lab-frame packet energy;
material exchange and deposition shall use the local-comoving convention.
Each solved step shall report initial and final retained radiation, source
injection, explicit deposition, escaped radiation, inner-boundary sink,
material internal energy, expansion work, radiation/matter frame-work
correction, and numerical residual. Gas `P dV` work and frame/work correction
shall remain separate.

The first transient observable shall use real escaped packets and retain exact
lab escape time, position, direction, frequency, energy, emission time, and
available metadata. Products shall be spectra and light curves indexed by
escape time. Observer arrival-time, distance, cosmological, and extinction
corrections shall remain external postprocessing.

## Implementation

1. Define typed transport tally objects for integrated paths, rates, deposited
   energy, escape, return, and uncertainty diagnostics.
2. Implement exact evolving-volume and local-comoving path weighting, with raw
   sums retained before division by time, volume, `4 pi`, or `h`.
3. Separate rate tallies from unnormalized interaction-energy sums used by the
   ledger.
4. Add global and per-shell energy ledgers and explicit residual diagnostics.
5. Add canonical escape records and derive escape-time spectra and
   bolometric/band light curves from them.

## Tests and validation

- Recover the analytic nonrelativistic `t^-3` volume integral and the static
  snapshot normalization limit.
- Verify unsplit/split path equality and time-weighted recombination of
  subinterval averages.
- Test full-relativistic radial and oblique path factors and exact-time line
  resonance rates.
- Verify transparent source, retained-census, escape, return, deposition, and
  material-energy ledger closure globally and per shell.
- Confirm that comoving-elastic interactions do not change material internal
  energy and that missing channels fail loudly.
- Compare escape-time products with the canonical escape records, not a second
  independent accounting path.

## Backward compatibility

Snapshot estimator normalization and snapshot spectra shall remain unchanged
outside transient mode. Transient outputs shall be explicitly labeled
escape-time quantities and shall not be presented as observer-time products.

## Alternatives

Applying a midpoint or representative volume after accumulating a path sum is
not acceptable because it loses the homologous time dependence. Reconstructing
the ledger from averaged rates is also rejected; realized packet and material
energy sums must remain independently available.

