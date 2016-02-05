TEP005: Formal Integral Method for noise-free spectra
=====================================================

Status
======

**Discussion**

Responsible
===========

@unoebauer

Branches and Pull requests
==========================

N/A

Description
===========

The main drawback of Monte Carlo methods is the inevitable introduction of
stochastic fluctuations, i.e. noise. In the case of Tardis, this is particular
relevant for the synthetic spectrum, the main product of Tardis calculations:
to perform a detailed spectral analysis, the line features must be
distinguishable from the noise. Thus, devising and implementing algorithms to
improve the signal-to-noise ratio is a crucial part of the Tardis
development process.

Currently, the so-called virtual packet scheme (see `Long & Knigge 2002`_, `Sim et
al. 2010`_, `Kerzendorf & Sim 2014`_) is used in Tardis to improve the fidelity of
the calculated synthetic spectra. Spectra calculated with this scheme are still
subject to noise, but its main advantage lies in its applicability to cases
with complex line-interaction treatments (e.g.  the Macro Atom scheme).

In cases in which only the resonance-scattering or the downbranching line
interaction treatments are used, a compelling alternative to the virtual packet
scheme exists. This approach, relying on a formal integration of the source
function has been developed by `Lucy 1999`_ and produces noise-free spectra.

This formal-integral approach should be implemented into Tardis and made an
alternative to the virtual packet scheme, whenever the scatter or downbranch
line interaction modes are used.

.. _Lucy 1999: http://adsabs.harvard.edu/abs/1999A&A...345..211L
.. _Long & Knigge 2002: http://adsabs.harvard.edu/abs/2002ApJ...579..725L
.. _Sim et al. 2010: http://adsabs.harvard.edu/abs/2010MNRAS.404.1369S
.. _Kerzendorf & Sim 2014: http://adsabs.harvard.edu/abs/2014MNRAS.440..387K

Implementation
==============

* The main procedure, namely determining the line emissivities an performing
  the integral should be performed in the C-part of Tardis
* It should be activated through the configuration file
* Some safety checks should be in place, e.g. not allowing the parallel use of
  the formal integral and the virtual packet scheme or prohibiting the use of
  the formal integral approach in conjunction with the macroatom line
  interaction treatment

Backward compatibility
======================

Should not affect backwards compatibility. The virtual packet scheme will not
be affected by this.

Alternatives
============

Concerning the aim of the formal integral approach (improving the
signal-to-noise in the synthetic spectrum), an alternative is already in place,
namely the virtual packet scheme. However, for Tardis calculations using the
downbranching scheme, the formal integral approach should be superior.
