TEP002: Meaningful title
========================

Status
======

 **Discussion**

Responsible
===========

@unoebauer
@wkerzendorf

Branches and Pull requests
==========================

tardis-sn/tardis#435

Description
===========

Currently, Tardis has the capability to store a number of important properties
of the model in an HDF5 file. However, the hdf5 file is currently not
self-contained, i.e. some stored properties require information which is
currently not stored. For example:

* last_line_interaction_in_id stores the ids of the last line transition in
  which the packet was lastly absorbed. The details of this transition, i.e.
  wavelength, atom number, ionisation state, etc. are stored in the
  atomic_data.lines pandas DataFrame which is not stored

There are also many important model properties, which are particularly useful
for diagnostic post processing of the run but which are not stored.

* unfiltered packet information together with LF wavelength before and after
  the last interaction (needed to construct Kromer plots)
* transitions probabilities (as seen in Issue #455, these are quite important)

There are several data located in different parts of the code that warrant
storing. The following is a more or less complete list:

* Radial1Dmodel
* Plasma
* MontecarloRunner
* Simulation
* atomic data

We propose to be able to essentially store all relevant information in an HDF5
file. Due to size constraints we also propose to be able to somehow select what
data is stored.

Implementation
==============

More to come.

**Storing Mode**: Maybe hardcoded sets of e.g. "minimal" or "detailed".




Backward compatibility
======================

N/A

Alternatives
============

The user stores the relevant information manually (pickling, to text files,
with numpy.savez, using pandas.DataFrame.to_*, etc.). This is not ideal since
it is not always clear at runtime which information may be of interest in the
future.
