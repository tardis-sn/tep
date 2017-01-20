TEP007: Isotope handling
========================

Status
======

**Discussion**

Responsible
===========

@wkerzendorf, @unoebauer

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

A number of details deserve particular consideration:

* specific composition profile: how to separate elements and isotopes?
* how to interprete values for element abundances in cases in which elements 
  and isotope abundances are given: does the element abundance represent only
  how much stable material is present or does it represent the sum of stable
  and radioactive material?
    Example:
      Ni: 0.4
      Ni56: 0.3
* relation between composition and time since explosion, t_exp, specified in 
  configuration file: Is the specified composition valid for t=0 or for 
  t=t_exp?

Backward compatibility
======================

This TEP proposes an addon to the code and does not change backwards
compatibility. However, it has to be ensured that old input/configuration files
in which no radioactive isotope abundances are provided are still understood
and can be still used by TARDIS.

Alternatives
============

Currently None
