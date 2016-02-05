TEP004: TARDIS Atomic improvement
=================================

Status
======

**Discussion**

Responsible
===========

@wkerzendorf
@lukeshingles

Branches and Pull requests
==========================

https://github.com/tardis-sn/tardis/pull/300

Description
===========

Converting atomic data such as levels, ionization thresholds, transitions from
the different atomic databases to a format that TARDIS needs is a non-trivial
task. This task is currently performed by the support package `tardisatomic`. It
currently converts Kurucz CD23 (or website) atomic text files into a sqlite database
This database is then in various steps converted to the much needed HDF5 format
that TARDIS uses. In addition, `tardisatomic` can also read from the Chianti database
to amend and/or replace existing atomic data to make hybrid atomic datasets.

Currently, `tardisatomic` has no clear workflow how to create these atomic databases,
a codebase that essentially doesn't allow the addition of other atomic databases
without a detailed knowledge of the code (which essentially 1 person has
within the collaboration) and no formal tests or documentation. In addition,
`tardisatomic` data lacks various data that is needed in upcoming versions of
TARDIS. Finally, the onus of ingesting atomic data from a variety of sources
and preparing it to a data format is by no means unique to TARDIS. This implementation
aims to implement a framework that allows for output to a variety of atomic database
formats for codes used in the collaboration (e.g. ARTIS)

This suggests a change of the existing structures with more modularity in
mind, which should be possible with the current experience of the team.



Implementation
==============

The idea for an implementation is to have a framework that uses a SQL database
as its central storage medium. This SQL database is fed by input classes or libraries
that read from a variety of source material. Store it in a more or less common format
within the SQL database and then provide a variety of output scripts that read
from the database and convert it to the required output formats.

Database module implementation
==============================

We chose an SQL database as it makes it easy to search and filter the data. The
use of the SQLAlchemy object abstraction layer will make it also very easy to
interact with the database from other python programs and thus allow for easy
extensibility. Furthermore, with the use of SQLAlchemy we can switch to
different SQL dialects (such as using sqlite for an initial implementation and
then switching to Postgres for larger databases).

We are still in the process of iterating on this structure. Essentially
https://github.com/tardis-sn/tardisatomic/blob/master/tardisatomic/alchemy/base.py
already contains a preview of the proposed structure.

This documented will be updated with more implementation details as soon as work begins

Input/Output modules
====================

Input and output modules should be located within `tardisatomic/io` and should
have a separate package (e.g. directory for each of the sources). These packages
should contain the different scripts for download and parsing.

We envision to have a simple input workflow like the following:

1. Download from the source using a function
2. Parse the downloaded material to an pandas DataFrame
3. Ingest that Pandas Dataframe into the database

The output modules will likely be a bit more involved and there are currently
various ideas of how to implement those. Once the database has some data in it
it will be easier to judge how to proceed.

Sources
=======

We have identified a variety of sources for the different data:

** Atomic masses **

http://www.nist.gov/pml/data/comp.cfm
http://www.nndc.bnl.gov/nudat2/index.jsp

**Ionization Energies**

http://physics.nist.gov/PhysRefData/ASD/ionEnergy.html

**Levels & transitions**
http://kurucz.harvard.edu/atoms/1401/gf1401.gam
http://www.chiantidatabase.org/chianti_direct_data.html - works well with chiantipy

http://norad.astronomy.ohio-state.edu/
http://kookaburra.phyast.pitt.edu/hillier/web/CMFGEN.htm -(compilation derived from Kurucz, Nahar, others)


Backward compatibility
======================

We will likely need to change both `tardisatomic` and `tardis` in this process.
However, while the change to `tardisatomic` will be significant - the changes to `tardis`
will be minimal and can be provisionally implemented by having a compatibility
layer that will ensure compatibility with the existing `tardis` structures at runtime.

The new atomic data format which likely will carry more information as the current
one will only change to a more Pandas friendly way of storing, but will conceptually
stay the same.


Alternatives
============

The most obvious alternative is leaving the old system. However, multiple incidents
in the use of TARDIS have shown this to be more time consuming than to implement
this TEP. Especially, the poor legibility of the current code and the lack of
documentation for which data is stored in which units makes this code not only
an annoyance but also a potential pitfall for the scientific veracity of TARDIS.
