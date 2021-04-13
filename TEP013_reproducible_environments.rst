TEP013: Reproducible TARDIS environments
========================================

Status
======

- **Discussion**

Responsible
===========

- @epassaro
- ?
  
Branches and Pull requests
==========================

-

Description
===========

We use ``conda`` to install the packages required to make TARDIS work
on every platform (laptops, servers, CI services, etc.) using the 
so-called YAML environment files.

This approach has been useful for a long time but has some important
drawbacks, being the most notorious the solving of the environment file. 
Packages are being updated all the time in the ``conda-forge`` channel, 
making the environment extremely volatile: a working TARDIS environment 
could be solved different on the next hour despite not having make any 
changes on the YAML file.

This behavior could lead to a few known issues that almost every TARDIS
developer faced at least one time:

a) tests passing locally and failing on CI/CD
b) different test results for different users
c) impossibity to recreate old environments
d) more...

Environment reproducibility is not an exclusive problem of the TARDIS
community, currently there's a running discussion between many developers
about how to achieve this. On this TEP I propose to implement the
`recommended solution <https://docs.conda.io/projects/conda/en/master/user-guide/tasks/manage-environments.html#building-identical-conda-environments>` 
by the ``conda`` developers: the generation of ``spec``
files. The ``spec`` files contain the exact version of all the packages
installed on an environment, removing the dependency solving step. Then,
**we can unequivocally identify a TARDIS commit with a fixed environment.**

The reproducibility of TARDIS environments will make our development cycle 
more robust, allowing us to focus more on development than on fixing stuff.


Implementation
==============`

The steps required to make this TEP work are:

1. Make a new pipeline to periodically (weekly or monthly) solve the ``tardis_env3.yml`` environment file and run tests. If tests pass, dump the environment to the ``spec`` files (one per OS) by doing ``conda list --explicit`` and push these files to the repository. If tests fail, do nothing and wait for a fix on the `tardis_env3.yml` like we always do.
2. Adapt the already existing pipelines to use the ``spec`` files.
3. Update the installation guidelines.


Backward compatibility
======================

Naturally, we can't reproduce environments prior of the implemantation of this TEP.

Since we are not dropping the ``tardis_env3.yml`` file (we still need it to generate the ``spec`` files)
this installation method should not break anything.


Alternatives
============

`constructor <https://github.com/conda/constructor>` is an official package from the ``conda`` 
developers to distribute packaged environments. With ``constructor`` you can make your own 
Miniconda-like installer, shipping packages and installation scripts in a single binary file.

We can think about ``constructor`` as a complement of the ``spec`` files.
