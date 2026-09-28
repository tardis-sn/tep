TEP008: Tighter integration of Atomic Data, Plasma and Model
============================================================

Status
======
**Discussion**

Responsible
===========

@wkerzendorf
@yeganer

Branches and Pull requests
==========================

Description
===========

TARDIS current plasma is structured in a very modular way that allows easy
testing and quick addition of new plasma modules. We restructured the Plasma
to this new form in 2015 and implemented the most important parts of the
framework. Due to time constraints, not all functionality was moved to this new
structure and there is still considerable amount of legacy code. In addition,
while the functionality is there we lack documentation that explains the new
rationale to the collaboration and users.

With this TEP we have multiple aims - ensure that all remaining parts that will
fit within the new structure are moved. Ensure that we update the documentation
to better reflect the current state of the code. This will include several
examples on how to write new plasma modules.

Implementation
==============

Important steps for the implementation:

- Modify the framework created for the plasma part to work for the whole model.
  This framework should be it's own module and makes heavy use of networkx.

- Adapt the plasma part to use the new module

- Convert the procedural approach used when loading atomic data to use the new framework.

- Connect everything in a way such that the networkx graph represents the model.

Backward compatibility
======================

This move should not break backwards compatibility.

Alternatives
============

TBD
