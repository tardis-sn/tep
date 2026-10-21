# TEP023: Continuum-enabled nonhomologous Type IIP transport

## Summary

Enable the existing Type IIP continuum calculation to use independent radius
and velocity profiles and the existing nonhomologous Sobolev real-packet
transport. The first supported combination uses partial relativity, macro-atom
line interactions, the current IIP start-of-flight continuum-opacity
approximation, and the current IIP stationary-snapshot thermal balance.

This proposal starts from the separate IIP and nonhomologous modes on current
`upstream/master`. It defines the combined physics contract, its validity
limits, and the verification required before enabling it. The intended IIP
profiles must pass explicit checks on the velocity gradient, bound-free edge
crossings in both directions, and the error from freezing continuum opacity
over a packet flight. The first implementation retains that existing
approximation within a validated regime.

## Status

**Discussion.** This proposal is ready for review; implementation of the
combined mode has not begun.

## Responsible

@connor-mcclellan

## Pull requests and Issues

No pull request or issue has been opened for this TEP yet. Add the discussion
PR and subsequent implementation PRs here as they are opened.

Related existing TARDIS PRs are
[#3688](https://github.com/tardis-sn/tardis/pull/3688) for nonhomologous line
distance and Sobolev depth and
[#3689](https://github.com/tardis-sn/tardis/pull/3689) for independent CSVY
radius and velocity geometry.

## Related work

- Current-code baseline: TARDIS `upstream/master` at
  [`d98a982994`](https://github.com/tardis-sn/tardis/commit/d98a982994),
  2026-10-20. Paths below describe that snapshot and should be checked again
  during implementation.
- The current IIP workflow supplies continuum plasma, thermal balance, and
  continuum event physics; the current nonhomologous mode supplies explicit
  radial velocity interpolation and real-packet line tracing.
- The representative `rebin_mesa_data.py` and
  `christians_atomdata_converted_04Dec25.h5` were inspected as local inputs.
  The script is untracked in the TARDIS checkout, while the atomic file is
  tracked in `tardis-regression-data` at `a314fa8`. Publish a reproducible
  preprocessing recipe and record the model and atomic-data provenance with
  the validation results.

## Description

### Science requirement and expected behavior

Type IIP spectral analysis needs continuum absorption, emission, ionization,
and thermal balance together with a velocity field whose shell radii and
velocities are independent. A configured nonhomologous IIP model should:

1. accept explicit shell radius and velocity boundaries, retaining the
   existing piecewise-linear radial velocity interpolation;
2. solve the existing IIP continuum plasma and macro-atom problem using
   nonhomologous line optical-depth coefficients and directional escape
   probabilities, including within trial population solves;
3. transport real packets through the existing nonhomologous line-resonance
   sequence while allowing electron, bound-free, and free-free events;
4. accumulate and normalize the continuum estimators consumed by the IIP
   plasma and thermal-balance iteration;
5. use the local velocity at the event for packet-frame energy and frequency
   transforms, following the existing partial-relativity angular convention,
   rather than deriving velocity from `r / time_explosion`; and
6. produce an emitted real-packet Monte Carlo spectrum, with unsupported
   spectrum requests rejected before transport setup.

The feature is intentionally limited to the current IIP treatment of selected
continuum species, blackbody inner boundary, stationary hydrodynamic
snapshot, and macro-atom cooling channels. `time_explosion` remains the
physical epoch for density, isotope decay, and input-model semantics. It must
not stand in for a nonhomologous velocity gradient.

### Current gap

`tardis/workflows/nonhomologous_tardis_workflow.py` builds line-only plasma
and bound-bound macro atoms and has an explicit continuum TODO. Its plasma
factory rejects configured continuum species without `allow_continuum=True`.
`modes/nonhomologous/solver.py` constructs no continuum Numba state and
returns no continuum estimators. Its packet loop and tracer handle only lines
and electron scattering.

`tardis/workflows/type_iip_workflow.py` builds continuum plasma, opacity,
macro-atom state, and thermal balance, but its hydrogen NLTE Sobolev input
uses a coefficient proportional to `time_explosion`. The population residual
recomputes a homologous escape probability for each candidate state. Merely
replacing final packet opacity would leave the ionization and level
populations inconsistent with nonhomologous line transfer.

The IIP event functions use `r / time_explosion` as the local velocity.
Nonhomologous transport currently rejects full relativity. The partial IIP
configuration already selects a simple blackbody packet source; no source
velocity correction is required for this milestone. The two existing solver
modes therefore cannot be combined by selecting a different workflow class
or copying one packet loop.

### Representative IIP model input

The representative `rebin_mesa_data.py` interpolates MESA cell-center
velocities to outer radius edges, then interpolates density and abundances
onto a uniform velocity grid. The source snapshot also contains the
corresponding radius. The script currently omits radius from its binned
`structure.dat`. For the first milestone, write a CSVY model with independent
`radius` and `velocity` boundary columns, which the existing geometry parser
accepts, and aligned density and composition data. Adding a radius column to
the existing ASCII structure file alone would fail: its reader does not parse
that column and has a special first-density-row convention. The conversion
must explicitly preserve the reader's boundary and shell indexing, including
the unused first density entry where applicable. TARDIS then constructs
`Radial1DGeometry` without inferring `r = v time_explosion`.

Make the preprocessing reproducible: remove the script's user-specific
Matplotlib style and missing relative configuration dependency, preserve
elemental abundance when summing isotope contributions, and record the input
model, binning, and conversion command. Determine `time_explosion` from the
source model independently. The MESA header states 50 days after bolometric
maximum, while the separate synthetic `iip_config.yml` uses 16 days after
explosion; those are not interchangeable epochs. The representative run must
check ordered radius boundaries and alignment with velocity, density, and
abundances. The atomic HDF5 contains photoionization and macro-atom groups
and can serve as the existing IIP atomic-data input once its provenance is
recorded.

### Physical formulation retained in the first milestone

For a spherical radial flow, let a packet move a lab-frame distance `s` along
a straight ray. Its radius `r(s)` and radial direction cosine `mu(s)` vary
along the ray. The partial-relativity frequency treatment is

```text
nu_comoving(s) = nu_lab [1 - mu(s) v(r(s)) / c].
Q(r, mu) = d[v(r) mu]/ds
         = mu^2 dv/dr + (1 - mu^2) v/r.
```

At each line crossing, reuse the existing nonhomologous real-packet
resonance search, directional Sobolev depth, and line-estimator weighting.
The tracer's projected-gradient turning intervals remain in place, although
the positive-`Q` supported profiles do not exercise them. For line `j`, with
the existing atomic and population coefficient `A_j`,

```text
tau_line,j(r, mu) = A_j / |Q(r, mu)|.
```

Use the existing angle-averaged **local** directional escape probability in
the IIP rate and macro-atom calculations:

```text
beta_j(r) = (1/2) integral[-1,1]
            {1 - exp[-A_j/|Q(r, mu)|]} / [A_j/|Q(r, mu)|] dmu.
```

The present implementation evaluates this using a shell representative
velocity and gradient and angular quadrature. The proposal does not claim
that a local escape probability captures nonlocal coupling between distant
resonances in every nonmonotonic flow. It also does not regularize the
Sobolev singularity at `Q = 0`.

For continuum events, preserve IIP's current approximation **per packet
flight**: at the flight start, calculate `nu_comoving,0`, the selected
bound-free and free-free opacities, and their interaction-channel weights.
Hold them fixed until the next boundary or packet interaction. In the
current partial-relativity convention, continuous optical depth over a
distance `s` is `chi_0 s`, without the full-relativity Doppler path factor.
The nonhomologous tracer must combine this continuous depth with discrete
line jumps in increasing distance order. It draws one interaction optical
depth and selects the earliest event; a line crossing that does not interact
must leave the residual continuous optical depth intact. A new flight
re-evaluates continuum opacity in the new shell or after an interaction.

The existing IIP path estimators likewise use flight-start frequency, energy,
cross sections, and opacity over the traveled distance. This proposal retains
that approximation. It must use the resulting estimators in IIP's existing
normalization, photoionization, thermal-balance, and continuum-state update
sequence. It does not silently replace these estimators with a different
quadrature scheme in only one transport mode.

### Validity of the frozen-opacity approximation

In homologous expansion, comoving frequency decreases along a straight
flight. A nonhomologous flow can reverse this trend. If a ray starts below a
bound-free threshold and blueshifts above it before the next event, the
flight-start bound-free opacity is zero even though absorption becomes
possible later. Conversely, holding an above-edge opacity after a downward
crossing overestimates absorption. The IIP event path also uses the event
frequency for its bound-free energy split, so a frozen channel chosen above
an edge may become physically invalid below that edge. Photoionization
tables have upper frequency bounds that require the same audit.

**First-milestone decision:** support outward-moving profiles with `v/r > 0`
and `dv/dr > 0` throughout every active shell. This gives `Q > 0` for every
ray, prevents upward threshold crossings, and excludes projected-gradient
turning points and repeated line resonances from the supported regime. Check
both quantities on the radius-augmented binned product. Small fractional
departures from homology alone are insufficient: if
`v=(r/t)(1+epsilon(r))`, then
`t dv/dr=1+epsilon+r d(epsilon)/dr`, so a narrow wiggle can reverse the
gradient.

Positive `Q` does not prevent downward edge crossings or bound the change in
continuum opacity within a flight. Before accepting a representative model,
audit packet rays and frequencies against every selected bound-free
threshold and photoionization-table limit in both crossing directions.
Measure the change in continuous optical depth and event-channel weights
relative to a segmented frequency-dependent calculation, then propagate
that difference to continuum shape, hydrogen ionization, and selected line
profiles with a Monte Carlo error budget. Reject an invalid bound-free event
whose event frequency is below its selected threshold rather than allowing
an unphysical energy split. If crossings or opacity variation exceed the
agreed observable error budget, targeted edge handling is required before
that model is supported. Full frequency-dependent path integration remains
a separate enhancement.

The existing Sobolev requirement also remains: the line resonance length
`v_thermal/|Q|` must be small relative to the flow and plasma scales, and
continuum destruction across that length must be small or explicitly
validated for the selected IIP regime. In particular, a vanishing
`Q` does not have a valid first-order local Sobolev depth. The current
nonhomologous exception at exactly `Q = 0` is retained; the proposal makes
no claim for near-zero-gradient profiles without separate validation. For
specified line-forming shells, lines, and temperatures, the acceptance record
must include shell `dv/dr`, `v/r`, the minimum directional `|Q|`, the
**maximum** relevant `v_thermal/|Q|`, its ratio to velocity and plasma scale
lengths, and continuum absorption depth across that length. Refine shell
spacing to check the shell-representative escape probability, populations,
and selected spectrum diagnostics against packet-depth calculations at
resonance radii.

### Non-goals

- Frequency-dependent path integration of continuum opacity or estimators,
  general threshold-crossing treatment, and a replacement for the current
  IIP frozen-opacity approximation.
- Full-relativistic nonhomologous resonance, source, opacity, aberration, or
  estimator treatment.
- Time-dependent radiative transfer, ionization, density evolution, expansion
  work, or new adiabatic-cooling channels. The active IIP thermal-balance
  solver does not currently include an expansion-cooling term.
- Changes to atomic datasets, hydrogen Lyman-continuum suppression, the
  blackbody photosphere, or the selected IIP continuum species.
- Formal-integral continuum spectra or continuum-enabled virtual packets.
- Generalizing line transport near `Q = 0` or solving nonlocal generalized
  Sobolev coupling in the plasma rates.

## User-facing selection and compatibility

On current `upstream/master`, expose an explicit nonhomologous opt-in through
`TypeIIPWorkflow`. It accepts explicit radius
and velocity boundaries, uses the existing IIP continuum species and
macro-atom treatment, and selects the existing nonhomologous
partial-relativity transport conventions. Electron
scattering may be enabled or disabled. The selection must be explicit in
the workflow entry point or configuration; geometry type alone must not
silently change the physics mode.
Profiles that fail the positive-`Q` or continuum error-budget criteria are
outside the first supported regime.

Reuse the current IIP iteration sequence and the current nonhomologous
geometry and real-packet resonance logic through narrow adapters or hooks.
Extending the current nonhomologous solver with optional continuum state and
estimators is acceptable. A separate combined packet kernel may be used if
that keeps the existing line-only compiled signature stable. This proposal
does not require a new general transport capability system or a shared
solver refactor.

Combined-mode requests for virtual packets or formal-integral spectra must
fail explicitly before running packet transport. The supported spectrum is
the emitted real-packet Monte Carlo spectrum. Existing homologous IIP,
line-only nonhomologous, and classic calculations retain their prior
selection and outputs; their stored regression references are not changed
as part of enabling the new combination.

## Implementation

### 0. Produce a reproducible radius-aware model

Interpolate MESA edge radius onto the script's existing velocity bins and
emit a CSVY model with independent radius and velocity boundaries. Keep
density and composition aligned with the parser's shell convention, including
the unused first density entry where applicable. Remove local style and
configuration dependencies from the preprocessing recipe and sum isotope
contributions for each element. Record the source model, conversion command,
binning, independently established explosion epoch, and atomic-data commit.
Check units, shell ordering, and material alignment. No new rebinning
algorithm or ASCII geometry reader is required.

### 1. Plasma and opacity contract

Extend the IIP workflow's continuum-enabled plasma preparation to accept
`Radial1DGeometry` with explicit radii and velocities. Preserve its initial
populations, selected species, and snapshot thermal iteration. Refactor the
NLTE Sobolev input so the initial plasma seed, each candidate population
state, and each accepted state produce a line coefficient independent of
velocity gradient, then compute the appropriate directional, angle-averaged
`beta_sobolev` for each shell. The homologous limit must recover the existing
`A_j time_explosion` result. Use the same nonhomologous beta in macro-atom
transition probabilities and the same `A_j` in packet opacity, divided by
local `|Q|` at each resonance. Replace the initial IIP constructor's
homologous `time_explosion` Sobolev seed before its first transport iteration.
Normalize `J_blue` with the nonhomologous radiation-field solver's
`c/(4 pi time_of_simulation volume)` factor: its estimator already includes
the local `1/|Q|` contribution. Do not apply the IIP solver's extra
`time_explosion` factor. Route both the IIP convergence-estimate and
estimator-update call sites through this choice.

Build the existing `ContinuumMacroAtomSolver` and
`ContinuumOpacityState` from the accepted plasma state after every IIP
iteration. The accepted opacity state must carry the gradient-independent
`A_j` needed by the nonhomologous tracer alongside the IIP escape
probabilities and any representative shell depth; current IIP and
nonhomologous opacity solvers do not expose this combination through a
single call. The Numba opacity conversion must select continuum-enabled
macro-atom deactivation probabilities and keep its line and continuum
indices aligned. Guard `chi_e + chi_bf + chi_ff = 0` before forming any
continuum-channel probability in the new event tracer; the pre-existing IIP
bound-free zero-CDF edge case is separate
unless a combined-mode acceptance test exposes it.

### 2. Real-packet transport contract

Allocate continuum Numba state and thread-local continuum estimators in a
narrow combined compiled driver or an explicitly parameterized current
driver. Use the existing nonhomologous resonance sequence to locate discrete
line jumps, while continuous opacity competes for the same sampled
interaction depth. The first supported profiles have `Q > 0`; turning-point
and repeated-resonance behavior is outside this milestone. Retain IIP's
flight-start opacity and estimator inputs.
At a continuum event, choose electron, bound-free, or free-free with those
flight-start weights, activate the appropriate macro atom or thermal packet,
and emit using the event-position `geometry.get_velocity(r, shell)`.
Reject a selected bound-free event if its event-position frequency is below
the selected threshold; the model-level edge audit determines whether this
guard reveals a profile outside the supported regime.

Make local event velocity available to every line, electron-scattering,
free-bound, free-free, and macro-atom emission path. Preserve packet-energy
and tracker conventions, line cursor updates after continuous events, shell
crossing, seed order, and continuum-disabled reduction to line-only
nonhomologous transport. Keep the compiled driver in Numba nopython mode.

### 3. Workflow and spectrum handoff

Attach continuum estimators to transport state and feed them into the IIP
normalization, rate, and thermal-balance sequence on every iteration.
Use the existing partial-relativity blackbody packet source and explicit
inner radius. Validate unsupported virtual-packet and formal-integral
requests at setup. Document the accepted profile regime and the
frozen-opacity limitation in user documentation when the behavior lands.
Record the velocity and bound-free threshold regime, the opacity-variation
budget, and the Sobolev and shell-refinement results of the first validated
IIP profiles.

### Current-code integration points

The current nonhomologous `interaction_events.py` contains continuum-emission
helpers, but `interaction_event_callers.py` has no `continuum_event` or
continuum activation caller. Adapt the IIP continuum event and estimator
logic to the nonhomologous real-packet path, using the event-position
velocity instead of deriving `r / time_explosion`. The current nonhomologous
driver returns only bulk and line estimators; it needs thread-local continuum
estimators and a transport-state handoff. Its dormant continuum-emission
helpers expect continuum fields on the line opacity state, whereas current
IIP keeps them in a separate continuum state; adapt that state access before
enabling those helpers. The current IIP driver and event functions assume
homologous full-relativity kinematics and cannot simply be used unchanged.

The current nonhomologous virtual replay lacks continuum attenuation and
does not reproduce every real-packet line-resonance detail. This proposal
therefore keeps combined-mode virtual packets unsupported without depending
on virtual-packet relocation work.

## Tests and validation

First demonstrate the unsupported combined-mode selection as an
orchestration red test. Once the minimal interface exists, run each physical
test against the missing behavior and require it to fail for its stated
physical assertion. Characterize unaffected modes on the pre-change code.
State the independent oracle, regime, and tolerance in each test. The
following tests are acceptance claims, not a request to regenerate references:

| Test claim | Minimal regime and independent verification | Intended defect and tolerance basis | Evidence class |
| --- | --- | --- | --- |
| Binned radius reaches the combined workflow | Convert a small synthetic monotone `r(v)` model to CSVY with unequal `v` and `r/t_explosion`; independently compare parsed radius/velocity boundaries, aligned material arrays, and the first density-row convention to expected values. Re-run the published preprocessing recipe from its recorded inputs. | Detect discarded radius, wrong units, misaligned bins, or a non-reproducible input. Interpolation and floating-point error set tolerance. | Verification |
| Combined workflow constructs a continuum-enabled nonhomologous calculation | Explicit unequal `r/t_explosion` and `v` at shell edges; inspect plasma/opacity/transport state and run a packet. Separate structural assertions from numerical results. | Detect accidental homologous geometry, rejected species, missing continuum state, or incorrect workflow selection. Exact identity/shape checks only for wiring. | Orchestration |
| Candidate IIP populations use directional nonhomologous escape probabilities | One-shell toy atom with prescribed populations, positive `v/r` and `dv/dr`, and `dv/dr != v/r`; independently integrate `(1-exp(-A/abs(Q)))/(A/abs(Q))` over angle. Check the initial seed and each trial residual, not only final opacity. | Detect hidden `time_explosion` in the population solve or use of radial-only depth. Quadrature and floating-point error set tolerance. | Verification |
| Homologous Sobolev limit is recovered | Use `v(r)=r/t_explosion`; then `Q=1/t_explosion` for every direction and `tau=A t_explosion`. Compare beta and line-rate matrix with the existing IIP equations. | Detect wrong coefficient units, indexing, or angular weights. Floating-point/quadrature error sets tolerance. | Verification |
| Continuous opacity competes correctly with ordered lines | One expanding shell with `v(r)=a r`, `a>0`, prescribed frequency-independent `chi_e`, `chi_bf`, `chi_ff`, one isolated line, and controlled interaction-optical-depth draws. Compute continuous `chi_0 s` plus the analytic line jump. | Detect lost residual optical depth or wrong event ordering. Analytic distances permit near-machine-precision checks. | Verification |
| Continuum channels use local nonhomologous velocity | Force one electron scattering, one bound-free event, and one free-free emission at known event radii where `v(r) != r/t_explosion`; control or capture the sampled comoving emission frequency, then independently check the partial Doppler transforms of packet energy and emitted frequency. | Detect inherited IIP `r/t_explosion` use or wrong macro-atom channel. Floating-point transforms set tolerance. | Verification |
| Continuum estimators reach IIP plasma and thermal balance | Continuum-only kernel in a zero-velocity cell with prescribed cross sections and path length; calculate photoionization/heating/free-free estimator increments from the existing IIP formulae. Check both `J_blue` call sites against the nonhomologous normalization and inspect plasma and thermal states after two accepted iterations. | Detect missing or misindexed estimator handoff, double application of `time_explosion`, or feedback that stops after one iteration. Formula/solver tolerances set threshold; a mere finite-value assertion is insufficient. | Verification |
| Zero continuous opacity leaves line transport well defined | Set `chi_e = chi_bf = chi_ff = 0` with one finite-depth line and a controlled draw. Compare the resulting event and estimator values to the line-only oracle. | Detect division by zero when forming continuum-channel probabilities or an extra random draw. Exact event agreement is expected. | Verification |
| Disabled continuum reduces to the existing nonhomologous mode | Identical injected line opacity and line-only macro-atom state, packets, seeds, geometry, and electron data, with continuum processes and continuum macro-atom channels explicitly disabled. Compare real-packet outputs, line/bulk estimators, and tracker events. | Detect extra random draws or process events when disabled. Exact comparisons where the event stream is unchanged; otherwise justify roundoff. | Metamorphic verification |
| Representative IIP input lies in the stated validity regime | On the radius-augmented CSVY bins, independently compute active-shell `dv/dr`, endpoint `v/r`, minimum directional `Q`, and maximum relevant Sobolev length for specified lines and temperatures. Compare the latter with flow and plasma scales and calculate continuum depth across it. Audit rays against selected photoionization thresholds and table limits in both directions; compare frozen-flight optical depths and channel weights with segmented frequency-dependent values. | Detect negative or near-zero projected gradients, invalid Sobolev localization, downward-edge energy splits, or a material frozen-opacity error. Require positive sign margins and an agreed observable error budget, with Monte Carlo uncertainty stated. | Model validation |
| Shell and packet refinement preserve observables | Refine line-forming shells and packet count independently. Compare shell-averaged beta, accepted populations, continuum shape, hydrogen ionization, and selected line profiles, with uncertainty estimates. | Detect sensitivity to shell-representative escape probabilities or Monte Carlo noise. Tolerances follow the agreed model and atomic-data error budget. | Model validation |
| Unsupported outputs fail before transport | Request virtual packets or formal integral in the combined mode. Check precise early exception and no packet propagation. | Detect an unvalidated continuum spectrum path. Exact exception contract. | Orchestration |
| Compiled supported selections remain nopython | Execute a minimal packet for electron scattering on and off; inspect selected Numba driver/propagator signatures. | Detect object-mode fallback or wrong combined-driver selection. Exact compilation contract. | Compilation check |

Run existing nonhomologous transport and active IIP workflow tests with the
external `tardis-regression-data` repository before and after implementation.
The old `test_continuum.py` is explicitly skipped and is not evidence for
the new combination. A new combined-mode regression fixture must supplement,
not replace, the independent analytic tests; do not regenerate existing
references to make a new calculation pass. Compare continuum-only
transport kernels at zero velocity, where the partial and full frame formulas
coincide, or use a quantified low-`v/c` comparison with the IIP continuum-only
path. A finite-radius IIP workflow cannot use `v=0` at finite
`time_explosion`. Record reproducible seed, atomic data, model, and
regression-data commits.

For the first analysis models, record the analytic `Q > 0` condition over
active shells **and** a reproducible two-direction ray/frequency audit against
the selected bound-free thresholds and table limits. Record the maximum
relevant Sobolev scale, continuum destruction across it, shell and packet
refinement, independently established explosion epoch, and source and
atomic-data provenance. The existing `iip_config.yml` is a separate synthetic
example and is not a substitute for this representative model. These records
establish the frozen-opacity validity domain; they do not require a new
general runtime profile validator.

During implementation, run targeted tests and Ruff, the matching
regression-backed paths, the broader suite where feasible, and the
documentation build for the new user-facing guide. Do not infer validation
from a skipped test or use `--generate-reference` as a diagnostic shortcut.

Current pre-change characterization on `d98a982994`: the nonhomologous test
module passed 57 tests and the IIP workflow test module passed 6 tests with
the clean external regression-data checkout at `a314fa8`, using the local
`tardis-merger-workflow` Conda environment. These baselines cover the
separate existing modes, not the proposed combination.

## Backward compatibility

The proposal adds a previously rejected physics combination. It preserves
the established partial-relativity, start-of-flight opacity, snapshot
thermal, packet-seeding, and output conventions for existing calculations.
No existing regression reference should change merely because this
combination becomes selectable. The new combined result may differ from a
homologous IIP result because line depth, escape probability, and local
packet-frame velocity legitimately differ.

The first release must state that frozen continuum opacity is an
approximation and identify the validated profile regime. Users requesting
full relativity, virtual packets, or a formal-integral continuum spectrum
with nonhomology receive an explicit unsupported-combination error.

## Alternatives

- **Require full path-integrated continuum immediately:** deferred. It
  improves frequency-dependent opacity and estimators in IIP generally,
  but adds ray segmentation, quadrature, optical-depth inversion, and event
  CDF work beyond the initial combined-mode requirement. The first IIP
  analysis profiles must satisfy the two-direction edge and opacity-error
  budget; targeted edge handling is required before a model that fails that
  budget can be supported.
- **Copy the full IIP packet loop into a combined mode unchanged:** rejected.
  It assumes homologous resonance ordering, full relativity, and
  `r / time_explosion` event velocities. A narrow combined compiled kernel
  that reuses the current nonhomologous tracer is within scope.
- **Use `time_explosion` as a nonhomologous line gradient:** rejected because
  it gives wrong line optical depths, escape probabilities, and IIP
  populations whenever `dv/dr` and `v/r` differ.
- **Enable continuum virtual packets immediately:** rejected until virtual
  line replay matches the nonhomologous real-packet resonance sequence and
  continuum attenuation is implemented and tested.
- **Expansion cooling or time-dependent ionization:** separate scientific
  proposals. They may be important for Type II ejecta, but are not active
  IIP assumptions that can be silently carried into this first milestone.

## Acceptance criteria to settle during discussion

Before the status advances to Progress, agree on numerical error budgets for
continuum shape, hydrogen ionization, and selected line profiles. Use measured
Monte Carlo variation, shell and packet refinement, and model and atomic-data
uncertainty to set those budgets. The representative CSVY model, its physical
explosion epoch, and the two-direction edge audit must be reproducible; no
universal numerical tolerance is asserted for all Type IIP models.

## References

- Rybicki, G. B. & Hummer, D. G. (1978), [A generalization of the Sobolev
  method for flows with nonlocal radiative coupling](https://adsabs.harvard.edu/pdf/1978ApJ...219..654R).
- Lucy, L. B. (2002), [Monte Carlo transition probabilities](https://arxiv.org/abs/astro-ph/0107377).
- Lucy, L. B. (2003), [Monte Carlo transition probabilities II](https://arxiv.org/abs/astro-ph/0303202).
- Dessart, L. & Hillier, D. J. (2007), [Time-dependence effects in
  photospheric-phase Type II supernova spectra](https://arxiv.org/abs/astro-ph/0610136),
  for context on the deferred time-dependent problem.
