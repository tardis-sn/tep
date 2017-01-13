TEP010: Consistent Treatment of relativistic effects
====================================================

Status
======

- **Discussion**:

Responsible
===========

@unoebauer, @ssim
=======
@chvogl, @unoebauer, @ssim

Branches and Pull requests
==========================

Related issue: #448

Description
===========

As many other Monte Carlo-based radiative transfer approaches, TARDIS takes a
so-called mixed-frame approach to the problem. This means, that the packet
propagation is carried out in the so-called lab frame (LF) since distances and
time intervals are easily determined here. Radiation-matter interactions,
however, are carried out in the so-called co-moving frame (CMF), which is a
locally defined reference frame, which is advected with the fluid flow.

Such a mixed-frame approach requires frequent transformations of important
packet properties (e.g. packet frequencies, energies, directions, etc.) between
the two frames. Transformation laws exists (see for example Mihalas & Mihalas
1984), but only a very small subset of those is currently taken into account in
TARDIS, namely:

* first-order Doppler effect for packet energies and frequencies

The following effects are neglected or only partially accounted for

* angle aberration
* transformation of opacity
* correct frame treatment of Monte Carlo estimators

The collaboration should carefully assess if the current treatment is
sufficient to guarantee satisfactory accuracy in the low and
mildly-relativistic regime and decide which relativistic effects should be
included in TARDIS.

Implementation
==============

@chvogl will lead the implementation of part or all of the relevant changes as
part of this Type IIP project.

Backward compatibility
======================

The proposed improvements will change the results of a TARDIS calculation. In most 
cases, these changes should be small since TARDIS is designed for the applications
in which relatistic effects are small or at most only important up to first order in
v/c.

Alternatives
============

Leave as is and accept potentially large errors in limiting cases.
