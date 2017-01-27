TEP001: Integration Test Suites
=============================

Status
======

**In Progress**

Responsible
===========

@wkerzendorf


Branches and Pull requests
==========================

None yet.

Description
===========

TARDIS testing can be separated into two categories: unit testing and
full code testing. While the unit testing can be done relatively quickly and is
able to run for almost every single commit, the full tests might take several hours
to complete. This is too resource intensive for our continuous integration service
TRAVIS and for swift development.

The major issues are:

1. allowing to create "slow" tests that are normally disabled
2. Running them at regular intervals
3. Monitoring them through a detailed output (webreport)
4. Implementing different sets of them (e.g. Paper I, Paper II, fast, ...)


Implementation
==============

We have implemented integration test suites within the py.test framework.
Currently, these are run at a regular interval on the group server
opensupernova.org. The web-reporting system is available at
http://opensupernova.org/~wkerzend/tardis-integration/doku.php.

We would like to expand the framework with the following features:
  *  different integration test suites should be defined.
  Our idea here is to have a set of tests which proceed relatively fast
  and can thus be executed frequently and another set of very expensive
  but very detailed and rigorous tests which will be repeated less often
  * currently we only have plots showing us the expected vs actual values for
  a very limited amount of TARDIS properties. We would like to expand this.git



Backward compatibility
======================

This should not affect backwards compatibility

Alternatives
============

N/A
