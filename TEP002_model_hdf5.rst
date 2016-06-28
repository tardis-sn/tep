TEP002: TARDIS hdf5 data storage capabilities
=============================================

Status
======

 **Completed**

Responsible
===========

@ftsamis
@unoebauer
@wkerzendorf


Branches and Pull requests
==========================

tardis-sn/tardis#435

Final PR: tardis-sn/tardis#606

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

Reading simulation states is a future goal but out-of-scope for this TEP.
A suggestion for adding the ability to read simulation state is to use
`classmethods` for constructing the individual objects that are likely also
used in TEP006 and can be implemented after TEP006 is implemented.

Implementation
==============

Hints of this implementation have existed for a long time within TARDIS. The
general idea is to have each piece of information in an object with a `to_hdf`
method that mirrors
(http://pandas.pydata.org/pandas-docs/version/0.17.1/generated/pandas.DataFrame.to_hdf.html)
All information is stored in an HDFStore in a pandas specific format. This ensures
that ordering, column names, indices, etc. stay the same and makes it trivial
to access the information. Data that are not in array form or simple array form
should likely be converted to Pandas DataFrames or Series to be stored within the
HDFStore. An alternative might be a pickling or json object that is then stored
as a binary table within the HDF5 tree (maybe for dictionaries).

the `to_hdf` method should take at least two arguments: 1) an HDFStore object
and 2) a path to be stored within the hierarchical HDF5 file. This means that
for level population the path might look like the following:
`model001/plasma/level_population`.

The end goal would be to have nested functions that would start with the top
level `Simulation`-object and would propagate downwards into the hierarchical
data structure within tardis (e.g. `Simulation.to_hdf` calls the
`LegacyPlasma.to_hdf` and in turn calls the `PlasmaProperty.to_hdf`). This will
also allow the higher level `.to_hdf`-methods to select specific information to
store.

This might be grouped in hardcoded sets like  "minimal" or "detailed".




Backward compatibility
======================

N/A

Alternatives
============

The user stores the relevant information manually (pickling, to text files,
with numpy.savez, using pandas.DataFrame.to_*, etc.). This is not ideal since
it is not always clear at runtime which information may be of interest in the
future.
