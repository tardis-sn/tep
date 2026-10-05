# TEP020: Time-dependent plasma and matter energy

## Summary

This TEP defines the first-release kinetic ionization state, statistical
excitation treatment, thermal equation, deposition interface, and endpoint
solve for evolving matter.

## Status

**Discussion**

## Responsible

TBD

## Pull requests and Issues

None yet.

## Description

Current TARDIS solves equilibrium plasma and instantaneous thermal balance at
one epoch. It does not retain a solved physical plasma state or advance matter
internal energy. The first transient release shall evolve ion stages while
retaining the existing statistical-equilibrium treatment within each ion.

The primary plasma state shall be mass-normalized ion abundance
`Y_j = m_u n_j / rho`, together with isotope/element inventory reference,
solution time, derived ion fractions and electron density, level populations,
material internal energy, shared matter/electron/heavy-particle temperature,
and validity diagnostics. Homologous dilution shall therefore arise through
derived density, without an explicit dilution term in the abundance equation.

A fresh run shall initialize at `t_0` with the existing Type IIP equilibrium
and thermal workflow. Subsequent steps shall use backward Euler with
time-step-averaged radiation rates and endpoint candidate populations,
temperature, and electron density. The ordinary supported radiative and
collisional processes shall provide a conservative ion-transition generator.
Unsupported process or atomic-rate coverage shall fail validation rather than
fall back silently to equilibrium.

Level populations shall remain algebraic statistical-equilibrium quantities in
the first release. Helium Spencer–Fano nonthermal ionization and excitation
shall enter through one typed channel bundle with explicit units, indices,
atomic-data provenance, and diagnostic powers. The bundle shall be reevaluated
for endpoint trial state, while the deposited energy for the step remains fixed.

Matter shall obey a first-law update including radiation exchange, optional
deposition, and homologous expansion work. Stored energy shall include
translational, ionization, and excitation energy. Electrons and heavy particles
shall share one Maxwellian matter temperature initially; temperature shall be
recovered from the candidate internal energy and populations, and invalid
translational-energy remainders shall reject the trial.

## Implementation

1. Add a checkpointable mass-normalized kinetic state distinct from existing
   nonlinear-iteration `previous_*` values.
2. Adapt existing equilibrium rate generators without using rows overwritten
   by normalization constraints.
3. Implement backward-Euler ion evolution, abundance/charge/level closures,
   and endpoint temperature inversion.
4. Add the typed helium Spencer–Fano adapter and ensure channel powers and
   rates are neither missing nor counted twice.
5. Implement stored translational, ionization, and excitation energy, optional
   local-comoving deposition, and homologous `P dV` work.
6. Expose convergence, algebraic residual, channel, and energy diagnostics to
   the state commit checks.

## Tests and validation

- Verify positivity, elemental sums, ion-fraction normalization, statistical
  equilibrium, charge closure, and scaled nonlinear residuals.
- Verify a source-free homologous ideal gas follows `T proportional to t^-2`.
- Verify EOS inversion round trips and conservation with expansion disabled.
- Verify ionization and excitation energy changes are represented exactly once.
- Verify zero deposition is an identity case and duplicate or unmodelled
  deposition channels reject the trial.
- Recover equilibrium thermal balance in the fast-relaxation or large-time
  limit and compare timestep refinement against first-order behavior.

## Backward compatibility

The existing equilibrium plasma and snapshot thermal-balance workflow shall
remain the default. Separate electron/heavy-particle temperatures, delayed
nonthermal reservoirs, full level kinetics, and non-helium nonthermal channels
are outside this TEP’s first release.

## Alternatives

Evolving absolute number densities would require an explicit expansion dilution
term and makes decay daughters and checkpoint normalization more fragile.
Evolving every atomic level is deferred because ion-stage kinetics with
statistical excitation is the smallest model that captures ionization
freeze-out and can be validated against the existing solver.

