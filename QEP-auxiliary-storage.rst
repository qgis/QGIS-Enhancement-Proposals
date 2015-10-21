.. _qep#[.#]:

========================================================================
QGIS Enhancement ??: Auxiliary storage
========================================================================

:Date: October 2015
:Author: Hugo Mercier
:Contact: hugo dot mercier at oslandia dot com
:Last Edited: 
:Status:  
:Version:

Summary
-------

This proposal is about an evolution of QGIS aiming at storing auxiliary data
in a layer. Such auxiliary data are data used mostly for the needs of QGIS (symbology) and have no real
interest in being stored with the native raw geospatial data.

The need arises from the restrictions existing in the manual placement of labels.
Manual placement of labels are possible in QGIS by setting some labeling properties (X and Y position,
and rotation angle optionaly) as being "data-defined", meaning that values come from a column (or an expression).
But setting this up on an existing layer requires either to add new columns to the source layer, while it is
not always possible or desirable. Creating another table and using joins to it is possible but the synchronisation
between the two tables is not possible, leading to a solution not satisfactory.

We propose here a more integrated solution where the creation of auxiliary columns or layers
are brought more transparently to the user by storing them in a central repository carried by the
project file.

Requirements
------------

The proposed solution have the following requirements :

- It should allow to transparently add columns of any type to a layer, without adding them to the source data source.
Beside usual types (string, real, integer), we may also think of geometry types. We only consider types that can be converted to atomic types (string for WKT or WKB)
The user should be able to edit values in these columns when the layer is in editing mode.

- These values should not be stored with the source data, and should be carried by the project, i.e.
when the project file is moved or copied, the auxiliary data should still be there.

- Adding a new row to the layer also adds a new row for each auxiliary field (with a null or default value)

- Removing a row from the layer also removes the corresponding row of each auxiliary field

- New GUI elements should ease the creation of auxiliary fields
  - on each "data-defined" property
  - possibly on groups of "data-defined": for manual label placement or label connectors

- It should be possible to save auxiliary data to a new layer

Proposed solution
-----------------

We propose to add a new type of field, close to the existing "virtual field" (where a field can be defined as an expression).
Here a new type of field, "auxiliary field", will point to a central data storage.

The central data storage should allow to store any kind of auxiliary column definition and data for any layer. And it should
be linked to, or be part of, a project file. The access to auxiliary data should be efficient.

Core changes
------------

Project file
------------

The current project file format (XML) is too limited to carry an efficient storage of such auxiliary data. It should somehow be evolved
to include a binary indexable format. SQLite is chosen as a first candidate for this data format. But it might evolve if performance issues
are found.

Given the current state of the core code where lots of classes assume the project file is an XML file (everything that is saved to the project file
has a writeXML method), it seems hard to completely refactor the project file format to a format based on SQLite.

We propose to use a Zip file as the root of the new project file format. A Zip file exposes a filesystem-like structure.
The original XML-based project file will then be embedded in the Zip file. And additional files could de added, and in particular
an SQLite file.
It only adds a step after actually saving the XML tree: including it in a Zip file. Symetrically during the loading, the first step would be to extract the XML file and pass it to the
existing readXML methods.

Auxiliary fields
-------------------

An auxiliary field is defined for a given layer. It is a new type of field: a new enum value for QgsFields::FieldOrigin (OriginAuxiliary). The code
of QgsVectorLayer, QgsVectorLayerFeatureIterator and QgsVectorLayerEditBuffer will be changed to reflect this new type, so that they will appear in the list of fields of a layer (alongside native fields and virtual fields).

Since an auxiliary field is conceptually a (left) join on an external database table, auxiliary fields may only be added to vector layers that have a **primary key**.

Auxiliary fields are defined as a table in the central SQLite database.

For instance, for a vector layer L with columns (ID, A, B, AF1, AF2) with native fields ID (primary key), A, B and auxiliary fields AF1 and AF2, there is
an auxiliary table T with columns (ID, AF1, AF2). The auxiliary table is named after the name of the origin layer.

A new row is added to the auxiliary table when a new row has been added to the origin vector layer and a data has been set (not null).

A row is removed from the auxiliary table when the corresponding row has been removed from the origin vector layer.

An auxiliary table is added when the first auxiliary field of a layer is created.

An auxiliary table is removed and data are lost when the origin layer is closed. The closing of a layer should explicitly warn the user about the loss of auxiliary data.
An auxiliary table is also removed when every auxiliary fields of a layer are removed.

Auxiliary field as cache for expression evaluation
--------------------------------------------------

For data providers that do not support on-the-fly computation of values, virtual fields allow to augment a virtual layer by a field computed by an expression.

For some particular use cases (computing geometries by expressions), such computations may be slow. Evaluating the expression that defines a virtual field is needed only when
the fields it refers to change. We then propose to allow auxiliary fields to store values that are initially defined as expressions.

An expression-defined auxiliary field can only be accessed for reading. The underlying fields refered to by the expression will be listed and when attributes of the vector layer
is updated, linked expression-defined auxiliary fields will be updated.

Custom properties
-----------------

The auxiliary database could optionaly be used to store custom properties of the project, since it may be more suited than XML for the storage of large data, like geometries.

GUI Changes
-----------

Auxiliary fields will be represented in the list of fields by a distinctive icon. And new icons in the layer properties, as well as in the attribute table widgets will allow to create
or remove auxiliary fields.

Data-defined properties are the most probable place where auxiliary fields will be used.
In the menu that can be found for each data-defined property, a new entry could be added to allow the creation, in a click, of an auxiliary field for that property
and link it as the source of the data-defined property.

Similar GUI shortcuts could be added in order to automate the creation of a group of data-defined properties.
A button (found both in the labeling properties dialog of the layer and in the manual labeling toolbox) will be added for the enabling of manual placement of labels.

It will be possible to save auxiliary data of a layer to a plain vector layer, by selection of a set of auxiliary fields in the attribute table or layer's properties dialog.


Performance Implications
------------------------

Access to auxiliary data will take place during the retrieval of a feature, through a QgsVectorLayerFeatureIterator, if such fields have been selected.
The basic implementation will retrieve a row of auxiliary data for each feature retrieved (SELECT WHERE id = ). Since the central SQLite database
will be configured to use an index for each auxiliary table, speed should not be a major concern.

Faster access could be investigated when the QgsFeatureRequest is not filtered or if it uses a FilterFid or FilterFids.

Test Coverage
-------------

Core changes will be covered by unit tests: modifications to QgsFields, QgsVectorLayer and QgsVectorLayerFeatureIterator


Backwards Compatibility
-----------------------

A new format for project files is introduced here. Project files from previous versions will still be available for opening. When saving back these project files, they will be
saved in the new file format.

Other excluded approaches
-------------------------

Other approaches have been investigating and excluded. They may be reconsidered if circumventing their drawbacks is possible.

We may think of using table joins to handle such auxiliary data and ease the creation of such joins for the end user.
However, the current implementation of joins are too restricted : no editing of a joined value is possible and there is no
synchronization between the main table and the joined table.

Virtual layers come also in mind for the implementation of such feature (not yet integrated). This will require the implementation of virtual layers
to have a write support (through triggers) to offer editing possibility for auxiliary data.
Another concern about virtual layers may be about performances since in that model, values from the original data source would be
converted into QgsFeatures, then into a representation suitable for an SQLite virtual table and then to QgsFeatures. Such transformation
is not needed and an optimisation would be desirable.

Voting History
--------------

