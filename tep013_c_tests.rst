TEP013: Improving c tests
========================

Status
======

TEPs go through a number of phases in their lifetime:

- **Discussion**: The TEP is being actively discussed and is
  being improved by its author.  The mailing list
  discussion of the TEP should include the TEP number (TEPxxx) in the
  subject line so they can be easily related to the TEP.

- **Progress**: Consensus was reached on the mailing list and
  implementation work has begun.

- **Completed**: The implementation has been merged into master.

- **Superseded**: This TEP has been abandoned in favor of another
  approach.

Responsible
===========

@wkerzendorf

Description
===========

Tardis is a simulation code, therefore testing is important. The montecarlo radiative transfer part of the code
is written in C and requires the same care regarding tests. We use pytest for python and we want all tests to run
by doing 'setup.py test'. The current framework solves this by exposing the C structures to python, which requires to
keep a python version of some of the C headers updated. Because the tests are written in python and execute the library
functions, we are unable to mock functions which prevents the writing of tests for some functions.

This TEP proposes to use a testing framework for C/C++ that is supported by the pytest-c plugin. This would require to
rewrite the current c-tests in C and add tests for functions that previouslt were not testable.


Implementation
==============


1. Move the c-tests from python to C and run it from pytest
2. Make execution of c-tests optional (only if pytest-c is is available)
3. Add tests for previously untested functions

Backward compatibility
======================

With the new framework in place, the current c tests would be removed.

Alternatives
============

Leave the current framework in place. This limits design space when writing future c tests.
