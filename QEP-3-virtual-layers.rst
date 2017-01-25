.. _qep#[.#]:

========================================================================
QGIS Enhancement #: Support Of Virtual Layers
========================================================================

:Date: 2014/09/30
:Author: Hugo Mercier
:Contact: hugo dot mercier at oslandia dot com
:Last Edited: 2014/09/30
:Status:  Draft
:Version: QGIS 2.5+
:Sponsor: *To Be Confirmed* MEDDE, France - "Ministry of sustainable development" for some parts
:Sponsor URL: http://www.developpement-durable.gouv.fr

.. note::

    See :ref:`QEP 1 <qep1>` for description of QEP process.

# Summary
----------

This enhancement proposal is about the addition of "virtual layers" to the QGIS core.

A virtual layer is defined to be very close to what is called a "view" in the context of relational database
management systems. It aims at offering a formatted view of pre-existing data, usually without copy of data involved.
A view can also bring new data computed on existing data, but without the need to store them explicitly.

Even if it comes from the database world, this feature does not focus on any particular data provider. It is not restricted
to data providers that connect to databases.

The list of data representation and manipulation features are growing in QGIS. Such features usually already exists in database
engines and are reproduced in QGIS to circumvent the lack of these functionnalities for popular data formats (shapefiles, text-based, etc.).
But supporting advanced database features by coding them into QGIS is probably not a good idea if one wants to keep a light and robust code base.

This is why we propose to embed a database engine. Once existing layers of a QGIS project can be exposed to this database engine, new interesting functionnalities can be easily developed. It could also be used to refactor existing QGIS code that deals with database-oriented features.

For now the virtual layer is mainly seen for vector layers, but extension to raster layers could be considered in the future.

#. Possible use cases
---------------------

The proposed design is based on these different use cases for vector layers:

-   create a dynamic point layer based on X,Y(,Z) coordinates
-   allow to filter features based on a SQL query, even for data sources not designed for that (CSV, ODS, etc.), and including spatial operators (Transform, Intersects, etc.)
-   allow to join different layers (including with a spatial clause)
-   give access to attributes that are computed, based on the value of other attributes (this may be somehow redundant with the possibilities offered by virtual fields in expressions)

#. Proposed Solution
--------------------

SQLite provides a very useful feature designed to embed its database engine in a third party application: `virtual table`_.
It offers the ability to expose internal data as an SQLite table. Then any operations available on a regular table can also be applied to a virtual table. The implementation can then choose to apply or ignore some of the operations. 
A C API (as well as a `Python API`_) allows to create such a virtual table mechanism.

It makes a perfect candidate for the implementation of virtual layers in QGIS:

*   the library is open source
*   Spatialite, as an extension to SQlite is already used as a spatial format, with growing support in GIS applications
*   it brings an embeddable powerful SQL engine

This proposal is inspired by the `VirtualOGR`_ driver for Spatialite that allows to open any OGR-supported format as a virtual table.


.. _virtual table: http://www.sqlite.org/vtab.html
.. _VirtualOGR: https://www.gaia-gis.it/fossil/libspatialite/wiki?name=VirtualOGR
.. _Python API: https://github.com/rogerbinns/apsw

#.# Data model
-------------

A virtual layer in QGIS would thus be defined as a new type of (vector) layer with the following parameters:

*   a list of QGIS layers that are used as primary sources of data (including other virtual layers)
*   an SQL query for accessing data of primary sources
*   an optional list of types for each column of the virtual layer
*   an optional SQL query for updating data of primary sources
*   an optional SQL query for inserting data into primary sources
*   an optional SQL query for deleting data from primary sources

#.# References to data sources
------------------------------

Each data source is refered by a pair (provider name, URI).

Opening / creating a virtual layer will try to open each referenced layers.

#.# Type handling
-----------------

SQLite uses `dynamic typing`_ for each value.
However, one may want to make sure a static type is ensured for columns of a virtual layer (for use in feature form widgets for instance).

Some additional information are thus needed to carry typing information for columns of a virtual layer. They have to be defined during the construction of a virtual layer.
When fetching data however, column values may be of a different type than the type declared (stored in a QVariant). Type casting has to be handled when setting the SQL query.
This is similar to what SQLite calls "type affinity": some type is defined for a column, but the actual type of a given row/column value may change.

Most of the time, the type of a column can be detected by using introspection abilities of SQLite (through PRAGMA table_info). If the type cannot be detected this way, the user may want to supply a type name at the construction of the virtual layer, or failback to a default type (TEXT).

.. _dynamic typing: http://www.sqlite.org/datatype3.html

#.# Indexes
-----------

Indexes for non-geometry columns will be requested transparently by underlying data providers of primary sources.

Spatial indexes will have to be stored into the Spatialite database through the use of CreateSpatialIndex.
This implies user-defined SQL queries must take care of using spatial indexes as it is done with Spatialite. With explicit reference to
the spatial index like in:

.. code-block:: SQL

    SELECT * 
    FROM t1, t2
    WHERE ST_CoveredBy(t1.geometry, t2.geometry) = 1 
      AND t1.ROWID IN (
        SELECT ROWID 
        FROM SpatialIndex
        WHERE f_table_name = 't1' 
            AND search_frame = t2.geometry);

For in-memory virtual layers, spatial indexes will be thus recreated for each loading.

#.# Serialization
-----------------

A virtual layer can be created in memory, through the use of the "memory" SQLite backend, or can be written to disk.
The definitions of in-memory virtual layers can then be written directly as an XML string to a QGIS project file.

For virtual layers written to disk, the file format used will be Spatialite, but a special case of a Spatialite database. In particular, a specific schema or set of conventions will be used to ensure a given virtual layer file is correctly defined, like the way Spatialite adds some particular metadata tables to a regular SQLite database.

#. Implementation Details
-------------------------

A new QgsVectorDataProvider will be developed to handle virtual layers.

* parameters of the creation (URI of sources) will be passed as an URI, using a separator that is not used by other provider URIs
* detail: should a new parameter be added to the QgsVectorLayer constructor (a map of settings) to avoid to find a new strange separator ? (since a virtual layer may reference other virtual layers, this can lead to deep levels of escape character sequences)

It will be based on the existing spatialite provider.

For the main methods of the QgsVectorDataProvider API, the following behaviour is planned:

*   **constructor**: the constructor will create/open a Spatialite database (either on disk or in memory). If creation parameters are given, they will be used for the creation of VIRTUAL TABLEs from QGIS layers. A VIEW will carry the SQL query for reading. Other optional SQL queries will be used as TRIGGER ON UPDATE, ON INSERT, etc.
*   **getFeatures()**: Execution of the SQL reading query
*   **fields()**: use of SQLite introspection to get column types, or user-defined column types if any
*   **addFeatures()**: Execution of the SQL INSERT TRIGGER. Field values that are passed to the function but not referenced in the TRIGGER will simply be ignored
*   **deleteFeatures()**: Execution of the SQL DELETE TRIGGER
*   **addAttributes()**: Used to modify the list of fields retrieved from the VIEW or to add virtual fields.
*   **changeAttributeValues()** / **changeGeoemtryValues**: Execution of the SQL UPDATE TRIGGER
*   **createSpatialIndex()**: call to Spatialite's CreateSpatialIndex
*   **createAttributeIndex()**: does nothing

SQL queries for updating / adding / deleting data could be set to some simple defaults for simple virtual layers.

In link with this provider, a SQLite extension module able to handle virtual layer will be developed

* offering a complete Spatialite geometric view from QGIS data sources implies to return a BLOB for geometries formatted with the internal Spatialite format for geometries. The Python API regarding virtual tables support is too limited to implement that for test purposes.

#.# Example(s)
..............

Using the simple interface described above, the new provider will execute something similar to the following commands:

.. code-block:: SQL

    CREATE VIRTUAL TABLE point_layer_vl USING QgsVirtualVectorLayer('ogr','/path/to/point_layer.shp');
    CREATE VIRTUAL TABLE polygon_layer_vl USING QgsVirtualVectorLayer('postgis',"'dbname='countries' port=5432 user='gis' srid=3857 type=POINT table="public"."countries" (geom) sql='");
    CREATE VIEW virtual_layer AS SELECT b.id, b.geometry where Contains(b.geom, a.geom) FROM point_layer_vl AS a, polygon_layer_vl AS b;
    INSERT INTO geometry_columns ...
    CREATE TRIGGER ... INSTEAD OF UPDATE OF ...


#.# Relation to expressions
...........................

Using the SQL language to create views on data might seem a bit counter-intuitive for end users, since they are used to filter data based on "expressions".
The syntax of QGIS expressions is the same as the syntax of the SQL WHERE clause. Some difference still exist regarding functions. SQLite offers mechanisms to create or overload user-defined functions. So that the virtual layer provider module can implement the same set of user-defined functions that the one found in QGIS expressions.

Then the SQL "dialect" of QGIS can be seen as an extension of QGIS expressions (dealing with SQL parsing, nested queries, CTE and so on), not something different.

#.# Python Bindings
...................

Creation and use of virtual layers will be available through a new QGIS data provider.
There is no particular additional Python bindings needed.

#.# User interface
..................

UI side, a first simple interface to the creation of a virtual layer will be provided.

.. image:: qep3_img/simple_spatial_layer.png
   
A new option will be added to automatically create a virtual layer for the list of selected layers (either by right click or via a menu entry).

Integration to the DB Manager is also planned, where one could drag and drop a QGIS layer to create a virtual layer out of it.

Virtual layers could be marked in the legend with some special icons (?)

#.# Affected Files
..................

(required if applicable)

#. Test Coverage
----------------

(required for technical solutions/changes if applicable)

#. Performance Implications
---------------------------

(required if applicable)

#. Further Considerations/Improvements
--------------------------------------

From a end-user point of view, a first concrete application of the virtual layer mechanism is planned regarding the ability to filter a layer that has some 'joins' defined. Since filtering is not supported for joined fields, a virtual layer will be transparently created in that case. This would allow the user to define SQL-WHERE filters on such joined layers.

Open discussions :

* Should the "joins" properties of a layer be replaced by the use of a virtual layer underneath ? (without changing the existing UI)
* Same question with "relations" ?
* More generally, with such a virtual layer mechanism in place, the existing code of data providers could also be ported to SQLite modules. Any data source would then be seen as a virtual layer and could be easily joined to other sources.
* Virtual layers are for now thought as views that do not store any data on their own. But since Spatialite allows to add tables and store data in them, a generalization of the concept could be discussed that would pave the way for a generic native format for QGIS.

#. Restrictions
---------------

(optional)

#. Backwards Compatibility
--------------------------

(required)

#. Documentation
----------------

(required if applicable)

#. Issue Tracking ID(s)
-----------------------

(required)

#. References
-------------

(optional)

#. Miscellaneous
----------------

(optional)

#. Voting History
-----------------

(required)
