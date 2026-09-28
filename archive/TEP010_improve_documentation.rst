TEP009: Improve Documentation
=============================

Status
======

**Discussion**

Responsible
===========

@unoebauer, @wkerzendorf

Branches and Pull requests
==========================

None

Description
===========

The documentation is incomplete, many is incomplete, many important aspects of
TARDIS are missing. To improve the TARDIS manual and to make it easier for new
users to understand and use this program, the following aspects should be
either better documented or their existing description expanded:

* reflecting inner boundary option
* adiabatic losses: illustrate and explain why the sum of backscattered and
  escaping luminosity is smaller than the injected one
* atom data sources: where do for example the zeta values come from
* ionization/excitation treatments: what are main different treatments, what is
  their physical meaning and who proposed them
* number of shells set up: there is always in the specific structure mode a
  "dummy" photospheric shell
* number of iterations set up: make clear that the last spectral run always
  counts as one iteration, i.e.
* git clean -dfx, git clean -dfxn: when and why to use these commands
* config validator
* configuration options: what are the different options that Tardis recognises

All these points should find their way into a coherent full documentation.
Currently, the documentation has the following structure.

* Using Tardis
  * Installation
  * Quickstart Guide
  * Ejecta Model Setup
  * TARDIS Configuration
  * Examples
  * Credit & Publication Policies
* Looking under the hood
  * Physics behind TARDIS
  * Monte Carlo Primer
* Developing Tardis
  * Reporting Issues
  * How to contribute
  * Running tests
* References
  * TARDIS Papers and useful references

Whether this structure is appropriate and if it should be changed should be
discussed by the collaboration here.

Implementation
==============

Will be specified once the (potential) new structure of the documentation has
been decided.

Backward compatibility
======================

N/A

Alternatives
============

None
