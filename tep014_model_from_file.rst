TEP014: Reading simulation from file
========================

Status
======

**Discussion**

Responsible
===========

@wkerzendorf

Description
===========

Since GSoC2016 it is possible to save the state of a model. The next step
is to be able to restore a Simulation object from a hdf file such that
it is identical to the python object used to produce the file.
Improving on this idea it should be possible to continue the simulation
or analyze the Simulation object in the GUI.
If some crucial information is missing from the file, the writing routine
should be updated.

Open discussion:
Store AtomData in file or require the same AtomData uuid for reading?

Implementation
==============

1. Add Simulation.from_hdf(...) which instantiates a Simulation and recursivly
load Model, Plasma and MonteCarloRunner from the file.
2. Write tests ensuring save/load consistency

Alternatives
============

There's no alternative 
