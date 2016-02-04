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

Database module
===============

We chose an SQL database as it makes it easy to search and filter the data. The
use of the SQLAlchemy object abstraction layer will make it also very easy to
interact with the database from other python programs and thus allow for easy
extensibility. Furthermore, with the use of SQLAlchemy we can switch to
different SQL dialects (such as using sqlite for an initial implementation and
then switching to Postgres for larger databases).

???? Describe structure and what exists ????



Input/Output modules
====================

??? Describe necessary atomic data -- in scope/out of scope discussion ???
??? Describe output formats ???


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
