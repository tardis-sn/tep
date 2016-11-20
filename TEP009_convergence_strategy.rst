TEP009: Convergence strategy object
===================================

Status
======

**Discussion**: The TEP is being actively discussed and is
  being improved by its author.  The mailing list
  discussion of the TEP should include the TEP number (TEP009) in the
  subject line so they can be easily related to the TEP.

Responsible
===========

@wkerzendorf @ftsamis

Branches and Pull requests
==========================

All development branches containing work on this TEP should be linked to from here.

All pull requests submitted relating to this TEP should be linked to
from here.  (A TEP does not need to be implemented in a single pull
request if it makes sense to implement it in discrete phases).

Description
===========

Currently the convergence strategy code is embodied inside the Simulation code
in a non-modular way. Adding a new convergence strategy is not straightforward.

Instead, a `ConvergenceStrategy` object is proposed which would encapsulate all
the convergence logic. Different strategies should then subclass this base class
and implement/extend it accordingly.

A `ConvergenceStrategy` instance should be passed to the `Simulation` object and
should also have access to it, so the former can change the state of the latter.
Specifically, `ConvergenceStrategy` should have access to **at least** the following
attributes: `model.t_rad`, `model.w`, `model.t_inner`, and their estimated equivalents
along with the current iteration count and the maximum iterations number.

The`ConvergenceStrategy` interface must have the following two public methods:

- `has_converged()`: Returns True or False depending on if the convergence criteria are met. All the convergence requirements (including hold_iterations for example), no matter how complex, should be checked in this method.
- `advance_simulation()`: Advances the `Simulation` object to the next state according to the convergence strategy. **OR** `get_next_state()`: Instead of modifying the `Simulation` object return the attributes that need to be changed.

Discussion is needed to decide which of the two `advance_simulation()` vs `get_next_state()` would be the most fitting.

The base class should also implement a `from_config` classmethod which accepts a `Configuration`
object and initializes the proper `ConvergenceStrategy` subclass according to a configuration object.


Implementation
==============

1. A base class `ConvergenceStrategy` should be created.
2. Subclasses of `ConvergenceStrategy` for existing convergence strategies should be created implementing all the required logic.
3. The `Simulation` object should be cleaned up from any convergence strategy code and adjusted to use the new `ConvergenceStrategy`.

Backward compatibility
======================

N/A

Alternatives
============

There are not any alternatives discussed at this point.
