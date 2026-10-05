# TEP018: Finite-flight optical transport and time-dependent sources

## Summary

This TEP adds resumable optical packets, moving-boundary event transport, and
a temperature-history-driven inner boundary source.

## Status

**Discussion**

## Responsible

TBD

## Pull requests and Issues

None yet.

## Description

Current optical packets are reconstructed from initial seeds and normally
terminate within one transport call. Geometry, resonance locations, and
transport time are effectively snapshot quantities. Time-dependent transport
requires packets to survive a step boundary without being mistaken for escaped
or absorbed packets.

Each packet shall retain physical and emission time, stable identity, lab-frame
position/direction/frequency/energy, cell and line cursors, source metadata,
status, and resumable RNG state. A packet retained at `t_(k+1)` shall resume
from its census state. The old unfinished optical-depth draw shall be discarded
after survival is established; the next step shall draw a fresh exponential
threshold from the continued random stream and recompute its coefficients.

Within a step, the implementation shall compete interaction, moving-boundary,
and time-boundary events. Homologous spherical boundaries shall be solved from
the packet and boundary trajectories, filter roots by the relative radial
direction, ignore tangent roots, and select the nearest valid root. A
geometry-scaled tolerance shall prevent packets from becoming stranded just
outside a boundary. Opacity and emissivity shall be built from `state[k]` and
held fixed over the interval, while geometry, velocity, frame transforms, and
packet clocks use exact event time.

The inner boundary shall accept a strictly increasing, finite, nonnegative
temperature history covering the simulation interval. TARDIS shall interpolate
temperature piecewise linearly and evaluate it at each exact emission time.
Gross outward blackbody injection shall be derived from the homologous radius
and temperature, integrated deterministically over each step, and sampled in
emission time from that same cumulative energy integral. A fresh run shall have
an empty initial census. Inward crossings shall terminate as returned energy,
not reflect or re-emit.

## Implementation

1. Extend packet storage with clocks, identity, census status, and RNG state.
2. Add a census/resume path that preserves line cursors and consumes no new
   interaction channel at the boundary.
3. Replace fixed-epoch event distances with exact-time moving-boundary and
   resonance calculations, including static-limit and near-boundary handling.
4. Keep first-release opacity piecewise fixed per step and document the
   instantaneous macro-atom approximation.
5. Add temperature-history validation, blackbody energy integration, and
   deterministic emission-time sampling.
6. Record gross injection, returned energy, and net boundary transfer in the
   time-step ledger.

## Tests and validation

- Verify analytic radial and oblique moving-boundary intersections and the
  static spherical limit.
- Verify that a time boundary wins over a later spatial event and that a
  boundary-stopped packet is neither escaped nor absorbed.
- Verify censused-packet resume, Sobolev cursor behavior, conditional
  exponential redraw, stable identity, and deterministic RNG continuation.
- Compare grey survival and resonance cases with analytic probabilities.
- Independently integrate the source history, test invariance to inserted
  transport boundaries, and verify zero-energy steps emit no packets.
- Reject incomplete, duplicate, nonmonotonic, nonfinite, or negative source
  history inputs.

## Backward compatibility

The existing snapshot packet path and source normalization shall remain
available when transient mode is not selected. Virtual packets and formal
integration shall not be silently treated as transient products; they remain
deferred until a causal history is available.

## Alternatives

Retaining unfinished optical depth across a step would couple a packet to old
opacity and is unnecessary: the exponential memoryless property makes a fresh
draw exact conditional on survival. Reusing a prescribed luminosity as the
source boundary is rejected because the accepted boundary condition is a
temperature history with derived gross emission.

