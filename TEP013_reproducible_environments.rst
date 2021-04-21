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

a) tests passing locally and failing on CI/CD.
b) different test results for different users.
c) impossibity to recreate old environments.
d) more...

Environment reproducibility is not an exclusive problem of the TARDIS
community, currently there's a running discussion between many developers
about how to achieve this. On this TEP I propose to implement the
`recommended solution <https://docs.conda.io/projects/conda/en/master/user-guide/tasks/manage-environments.html#building-identical-conda-environments>`_ 
by the ``conda`` developers: the generation of ``spec`` or ``lock``.
files. The ``spec`` files contain the exact version of all the packages
installed on an environment, removing the dependency solving step. Then,
**we can unequivocally identify a TARDIS commit with a fixed environment
that can be installed at any point in the future.**

The reproducibility of TARDIS environments will make our development
cycle more robust, allowing us to focus more on development than on
fixing stuff.

Also, these files let us to use `caching` on our pipelines, making them
faster.


Implementation
==============

The main drawback of the ``conda list --explicit`` command is it does not
recognize ``pip`` installed packages. A possible workaround is to separate
``pip`` dependencies in a separate file ``extra_requirements.txt``
and adapt the existing YAML recipe to use it::
    - pip:
        - r: file:extra_requirements.txt

Installing the environment with ``conda env create -f tardis_env.yml``
will work as usual.

To make a ``spec`` file we would need to make a new pipeline
to periodically solve ``tardis_env3.yml`` and install the environment
for each platform and run the tests.  If tests pass, dump the dependencies
to a ``tardis-{platform}.spec`` file by running ``conda list --explicit`` and
push them to the repository. Then, a reproducible environment is created by running:

.. code-block ::

  $ conda env create -f tardis-{platform}.spec
  $ pip install -r extra_requirements.txt

If tests fail by a package update, developers
and CI pipelines can keep working with the environment defined by the latest
``spec`` file, without stopping the development cycle. Meanwhile, the CI/CD 
team will try to fix the YAML file (like we already do!).

Backward compatibility
======================

Naturally, we can't reproduce environments prior of the implemantation of this TEP. But
since the old installation method is still available this feature should not break
anything.


Alternatives
============

- `constructor <https://github.com/conda/constructor>`_ is an official package from the ``conda`` 
  developers to distribute packaged environments. With ``constructor`` you can make your own 
  Miniconda-like installer, shipping packages and installation scripts in a single binary file.
  We can think about ``constructor`` as a complement of the ``spec`` files.
- ``conda env export`` is similar to ``conda list --explicit`` and recognizes installed ``pip``
  packages.
