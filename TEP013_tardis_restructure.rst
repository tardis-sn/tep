TEP013: TARDIS Code Restructure
===============================

Status
======
New structure currently being planned, and project will be assigned to a GSOC student.

Responsible
===========

* Marc Williamson
* Alice Harpole
* GSOC student

Branches and Pull requests
==========================


Description
===========

The overall goal is to restructure the TARDIS code to improve readability and maintainability, 
reduce code duplication and make it easier to extend existing functionality. We will use the 
Factory software engineering framework to reorganize TARDIS code into a more structured 
class-subclass relationship.

We suggest starting the restructure with the TARDIS model class, Radial1DModel. Instead of 
the current situation where Radial1DModel has multiple class methods for creating objects,
there should be multiple subclasses that handle the unique properties of each current 
class method. The super class Radial1DModel should handle all common features between the 
subclasses to eliminate code replication.  The purpose of this restructure is to hide away 
the complication of reading and constructing a model in child classes, while the rest of the 
TARDIS simulation only ever interacts with the parent Radial1DModel class.

Resources
=========

* Intro to UML diagrams: `Link <https://www.lucidchart.com/blog/types-of-UML-diagrams>`_
* TARDIS UML from hackathons: TBD

Starting Guide: Restructuring Radial1DModel
===========================================

There are two class methods, from_csvy() and from_config() that each create objects
of the Radial1DModel class, but each class method reads the initialization data from
different file formats.  

1. Break the current Radial1DModel into two classes, CSVYModel and ConfigModel, with
each class corresponding to one of the current class method initializations.

2. There is probably a lot of replicated code between the new CSVYModel and ConfigModel
classes. Move this replicated code into a superclass, InputModel.

3. The InputModel will probably need to keep track of which physical properties are specified
by the user. Consider how to implement this so that when the TARDIS simulation is running,
we can easily check what the user has specified.

4. Certain physical properties pertaining to the model are only calculated during the TARDIS
simulation, like the radiation temperature. Create a new TARDISModel class that has an InputModel
object as an attribute. The TARDISModel will handle storing model properties calculated by the
simulation.




