TEPXXX: Isotope handling
========================

Status
======

**Discussion**

Responsible
===========

@wkerzendorf

Branches and Pull requests
==========================

https://github.com/tardis-sn/tardis/pull/409

Description
===========

Decaying isotopes are an important part of supernova physics. Thus allowing for
specifying isotopes within TARDIS is a useful feature. Currently, some of this
is already implemented when reading ARTIS files, but only pertains to very
specific isotopes and with hardcoded numbers. In addition, the energy released
by the decay could be calculated and used in the TARDIS simulation which
currently it does not do.

We propose to include a method to specify and decay isotopes within the TARDIS
framework. This will include the internal code structure as well as the means to
specify those within the configuration file. Thus this TEP has some links to
the proposed changes in TEP006 as an implementation of TEP006 will make this
TEP easier.

Implementation
==============

There are already some implementation details given in PR409. This PR relies
heavily on the pyne package that can calculate the isotope abundances after
a given decay time.




Backward compatibility
======================

This TEP proposes an addon to the code and does not change backwards
compatibility.

Alternatives
============

Currently None
