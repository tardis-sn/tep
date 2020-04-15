===============================
TEPXXX: Optimization with numba
===============================

Status
======

**Discussion**

Responsible
===========

@harpolea @lukeshingles

Branches and Pull requests
==========================

[numba_montecarlo](https://github.com/tardis-sn/tardis/tree/numba_montecarlo)

#929

Description
===========

In order to optimize the execution of several computationally intensive parts of TARDIS, we currently use Cython. This converts these parts of the code to C and compiles them ahead of time, significantly improving their performance. However, while fast, Cython involves a lot of extra boilerplate code, making the code overall harder to follow and more difficult to maintain. We would, therefore, like to convert this Cython code to Numba, a just-in-time Python compiler that involves significantly less additional code to Cython but can achieve similar performance.

We would also like to explore further ways to optimize the code through e.g. restructuring.

Implementation
==============

1. Review the existing PR which converts the Monte Carlo routines to numba. If necessary, make modifications so that PR is ready to merge. 

2. Profile the code to identify bottlenecks. Attention should be paid to how the code is profiled (e.g. be careful profiling JIT compiled numba code) and how performance varies between different types of problems (e.g. do some types of problems spend much longer on specific tasks than others?).

3. Explore ways to eliminate bottlenecks. This could be through:
    - converting certain routines to numba, 
    - refactoring certain parts of the code to e.g. eliminate repeated operations and costly data accesses, 
    - and large scale restructuring. 
A number of different approaches should be explored and reviewed in order to find the optimal solution. 
For larger scale restructuring, this should be done in conjunction with the specific restructuring project.

