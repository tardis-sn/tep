TEP006: Configuration parsing improvement
=========================================

Status
======

**Discussion**

Responsible
===========

@wkerzendorf

Branches and Pull requests
==========================

N/A


Description
===========

TARDIS relies on the YAML format to store its input configuration. There is
a wide variety of options that can be configured. Ranging from the geometric
description of the model to switches for different modes of calculating the
plasma state. This fact is complicated that we aim to provide a multitude of
input formats. For example, one can specify a simple model with equally spaced
shells and abundances that are uniform throughout the model using one data
scheme. But also provide files that described a complex models with unevenly
spaced shells and different abundances for the same model. Depending on the
model, TARDIS needs to parse the information in different ways to convert
them to the common format needed within the code (DataFrames).

The current way the code deals with the input of configuration ways is divided
into several steps:

1. Parse the YAML file with PyYaml

2. Validate the data with a schema:
TARDIS validates the given information (e.g. that v_inner is given
as a quantity). TARDIS also uses the schema to complete information that is not
given in the current configuration file (e.g. default values). Thus the schema
also serves as an important document that will document the available options.
The current format of the schema and the validation code are an internal product,
however the lead developer of this project (@mklauser) has left the team since
and thus we have little expertise with this part of the code
(`tardis/io/config_validator`) anymore.

3. Transform the given information for the construction of model, plasma and
Montecarlo part. This is mainly done in `tardis/io/config_reader.py`. The code
is written in an awkward way that uses many if-statements in one long function.
It is not very extensible and hides important parts of the code. This module
also contains a long list of obsolete functions and is badly tested. A perfect
example for difficult to maintain code with only one developer having a better
insight.


Implementation
==============

The suggested end goal sees the configuration system in a more tested,
documented and extensible ways than it currently is. There are several major
goals:

1. Exchange the current `config_validator` with the readily available package
`jsonschema`. This will remove code from the codebase of TARDIS and thus make it
easier to maintain.

2. Use YAML tags to do some of the easy parsing.
One first step is to parse quantities like '5 km/s' using an implicit yaml tag

3. We suggest that we put the conversion of different input models, especially
for models in classmethod constructors for these classes. An alternative would be
to have some scheme where one could register converters. This part of the
TEP is currently the most uncertain and implementation attempts should look for
alternatives for this step.


Backward compatibility
======================

We introduced versioning of the config files in a previous TARDIS version.
This might be a point where we switch to a newer configuration scheme that will
be more powerful and easier to use. This might indicate a switch and previous
configuration files might then only be read with TARDIS version <= 1.5

Alternatives
============

There are several alternatives most notable in the third Goal and we suggest
researching the best course of action here. 
