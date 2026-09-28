TEP012: Overhauling the Qt-GUI 
==============================

Status
======

- **Discussion**:
This will be replaced by new jupyter widgets.

Responsible
===========

@unoebauer, @wkerzendorf

Issues, Branches and Pull requests
==================================

Issue `#690 <https://github.com/tardis-sn/tardis/issues/690>`

Description
===========

Tardis has a `Qt-GUI
<http://tardis.readthedocs.io/en/latest/gui.html#gui-explanation>` intended to
facilitate the analysis of TARDIS runs. At the moment, this GUI is not working
with current design of TARDIS. Fixing this problem should only be the starting
point of a general overhaul. In this process the following propositions should
be realised.

 * make simulation object sole GUI input
 * ensure that none of the current features are broken
 * merge tools from `tardisanalysis
   <https://github.com/tardis-sn/tardisanalysis>` into GUI (e.g. as additional
   tabs)
 * provide information about accessing the currently used simulation/model
   properties, e.g. in the form of a command prompt
 * (optional) provide a simple addon system which allows the user to easily add
   additional/own analysis functions

Eventually, the GUI should also be able to not only work interactively, but
also with stored models/simulation objects. This will require a helper which
builds the simulation object, used by the GUI, from the stored information.

Implementation
==============

The GUI should still be realised in QT.

Backward compatibility
======================

N/A since it is now broken.

Alternatives
============

Discontinue support of the TARDIS GUI.
