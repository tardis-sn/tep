TEP001: Extensive Test Suites
=============================

Status
======

**Discussion**

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

Ensuring to enable running of full-tests in a useful way this TEP aims to
work on two major issues:
1. allowing to create "slow" tests that are normally disabled
2a. finding a way to run those in regular intervals
2b. finding a way to couple the outcome of those to GitHub.

Implementation
==============

Our main testing system is py.test and we propose to keep these full tests also
in this test system.


1) We suggest that we mark extensive and slow tests using the py.test marking system
and skip them if no specific command-line flag is given.

An important point for some of these tests would be the production of comparison
plots. For now it is unclear how this would be facilitated, but maybe pytest-mpl
can be helpful in this regard.

2) For running this at a regular interval and inform us of any breakages, we
suggest running it on the group server opensupernova.org. The system that we might
be able to use is cron coupled with tox or direct py.test. On failure, we would
need to somehow alert the group of this.


3) A possible way to integrate these tests with GitHub is to use https://github.com/mikemcquaid/HookHand
this allows us to run a script and deliver the result back to github on any server

Extensive test suggestions
==========================

  - https://github.com/tardis-sn/tardis/pull/463


Backward compatibility
======================

This should not affect backwards compatibility

Alternatives
============

N/A
