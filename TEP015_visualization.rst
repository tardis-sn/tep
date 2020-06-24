TEP015: Monte Carlo Visualization
==================================

Status
======

**Progress**

Responsible
===========

@arjunsavel, @fcallan678, @jaladh-singhal, @ycamacho

Branches and Pull requests
==========================

Relevant PR: `TARDIS #1198 <https://github.com/tardis-sn/tardis/pull/1198>`_

Description
===========

This TEP aims to produce a tool for talks and demonstrations that visualizes the processes inherent to TARDIS. In particular, the core goal is to provide a self-consistent animation of packet propagation through TARDIS shells.

Currently, @arjunsavel and @finn have two independent packet visualizations. The former is static and logs the position and interaction of each packet as they move through TARDIS shells; the latter is a packet animation that has been used in multiple talks. These approaches will be merged (loosely speaking) into a single tool.


Implementation
==============

If “single_packet_loop” is logged by the log_decorator, the numba_montecarlo branch of TARDIS can track the position and interaction type of each packet at a given interaction. This information, dumped to a logging file, is subsequently parsed and plotted. Each interaction type is plotted as a different shape.
Animating this is the next step for this project. 
We are currently considering animating on the order of 2 dozen packets at a time, releasing further packets in blocks so that the animation does not become too cluttered. Packets within a block will be stepped through individual interactions simultaneously. Packets will be “grayed out” once they are lost to the photosphere to demonstrate that they are no longer being tracked.

Different shells will be color-coded by temperature; colormaps being considered include viridis, plasma, and the rainbow map employed by the older GUI. The photosphere will be colored darker to indicate that it is opaque.
Virtual packet capabilities will also be added at a later point.
Ideally, different TARDIS modes will be made more evident through this visualization, e.g. the difference between the nebular and opaque photosphere modes.
Once a working 2D version of this is produced in Matplotlib, further extensions include moving to a more robust library (e.g. d3.js) and toggling between 2D and 3D to demonstrate the physical nature of the TARDIS problem.


Backward compatibility
======================

All additions should be compatible with the `numba_montecarlo` branch.

Alternatives
============

N/A (as far as I know).