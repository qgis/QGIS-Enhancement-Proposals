.. _qep#[.#]:

========================================================================
QGIS Enhancement 13: Efficient Snapping and Geometry Queries
========================================================================

:Date: 2014/12/18
:Author: Martin Dobias
:Contact: wonder.sk at gmail dot com
:Last Edited: 2015/01/05
:Status:  Final Draft
:Version: QGIS 2.8

Summary
----------

This proposal is meant address several issues with snapping in QGIS:

#. Develop easier to use snapping API and clean up the snapping code
#. Allow querying to be more efficient with optimized data structures
#. Allow "point in polygon" queries for identify tool
#. GUI improvements to make snapping easier to use


Proposed Solution
--------------------

The current snapping implementation is spread across several classes (``QgsGeometry``, ``QgsVectorLayer``, ``QgsSnapper``,
``QgsMapCanvasSnapper``) and includes some extra features (like "excluded points" or mode for "topological editing")
as a core functionality.

Ideally the implementation should focus just on search for the closest vertices/segments
and leave extra features for higher level - to keep the querying core simple and make it easier to optimize it.

The aim is to simplify the implementation to the following structure:

1. **low-level querying** - for a set of geometries (typically one vector layer), find out the nearest
   vertex/segment or all near vertices/segments within tolerance.
2. **high-level querying** - operates on top of several layers, combines results from lower level
3. **extra functionality** - routines that are not directly related to querying, but are useful in context of snapping
   and are provided for user's convenience

Currently the snapping uses either simple geometry cache or iterates over layers if cache is not available.
The cache however does not have an efficient data structure for spatial querying (data are stored
in a map with feature IDs as keys and geometries as values). Introduction of R-trees should improve
the performance of queries even with greater amount of geometries and greater complexity of those (from
linear search to logarithmic). This should enable more interactive GUI functionality later,
such as mouse over effects for various tools or continuous mode for identify tool
(like Value Tool, not having to click on each feature).

Implementation Details
----------------------

1. new class ``QgsPointLocator`` [core lib]

   - operates on one vector layer, watches the layer for changes (added/changed/removed features)
   - defines ``Match`` class for retrieving results
   - builds data structures for efficient point location queries (R trees)
   - supports queries for matches within tolerance (vertex/edge), k-nearest neighbors (vertex/edge), point in polygon
   - supports reprojection (i.e. data may be indexed in different CRS)
   - supports lazy indexing (unless needed for a request, data are not indexes)
   - supports filtering of matches (e.g. accept matches only from a particular feature ID) with ``MatchFilter``
   - replacement for ``QgsSnapper``


2. new class ``QgsSnappingUtils`` [core lib]

   - keeps a cache of QgsPointLocator classes
   - snapToMap method: combine results from several QgsPointLocator instances according to setup
   - two modes for snapToMap
   
       1. snap to current layer (with default snapping type and tolerance)
       2. snap to all layers from configuration (advanced)
   - not directly connected to map canvas - need map settings and current layer to be explicitly set
   - optional snapping on intersections
   - client code can access locators
   - supports filtering of matches (e.g. accept matches only from a particular feature ID) with ``MatchFilter``
   - configuration can be read from project (but does not need to be)
   - replacement for ``QgsMapCanvasSnapper``


3. new class ``QgsMapCanvasSnappingUtils`` [gui lib]

   - convenience class derived from ``QgsSnappingUtils``
   - keeps settings always updated from canvas


4. new class ``QgsDigitizingUtils`` [core lib]

   - routines for topological editing


5. access to snapping utils in ``QgsMapCanvas``

   - new method ``snappingUtils()`` which returns associated snapping utils instance
   - new method ``setSnappingUtils()`` for association of snapping utils
   - main canvas in QGIS will come with pre-associated snapping utils that use the snapping config from QgsProject

The current snapping classes (``QgsSnapper``, ``QgsMapCanvasSnapper``) will continue to exist for API compatibility
until next major QGIS version, but QGIS code will not actively use them. With the new major version of QGIS
the class ``QgsGeometryCache`` (used by snapping code) will be removed. (Note: the ``QgsGeometryCache`` class is
used to speed up snapping by caching geometries in a map with feature IDs as keys and geometries as values.
The cache is used only when a layer is in editing mode. On every re-render of map canvas the cache is regenerated
based on current map view. It has very limitied use due to the used data structure and the way it is built.)

The increased performance of queries makes it possible to simplify the code. The current snapping implementation
recognizes few special modes (``SnapWithResultsWithinTolerances``, ``SnapWithResultsForSamePosition``) that are necessary
for extra functionality (topological editing, snapping on intersections) as they force the snapper to potentially
add further results used for processing later. The new implementation does not use such modes - instead it is
preferred to do another query when necessary for the processing. This keeps the code complexity low.


Indexing Strategy
-----------------

The plan is to index the whole layer. This can be done very efficiently with bulk loading of R-tree
(with STR algorithm) compared to interative insertions - and the resulting structure is also more efficient.

When layer is modified, the changes are detected with signal/slot mechanism and the indexes are updated.

For each type of query (vertex / edge / area) there is a separate R-tree for more efficient lookups.

- R-tree for vertices/edges stores individual points / edge's bounding boxes
- R-tree for areas stores bounding boxes of individual polygons and their GEOS geometry


Examples
--------

1. snap to a point according to project's snapping settings::

    m = iface.mapCanvas().snappingUtils().snapToMap(QgsPoint(11,22))
  
    if not m.isValid():
      print "no match!"
      return

    print "match: ", m.point(), m.distance(), m.layer(), m.featureId()


2. do queries on a particular layer::

      # get the point locator: uses map units
      locator = iface.mapCanvas().snappingUtils().locatorForLayer(layer)
      
      # find the nearest vertex and edge (no maximum tolerance)
      mV = locator.nearestVertex(QgsPoint(11,22))
      mE = locator.nearestEdge(QgsPoint(11,22))
      
      # find 5 nearest vertices and edges (no maximum tolerance)
      lstV = locator.nearestVertices(QgsPoint(11,22), 5)
      lstE = locator.nearestEdges(QgsPoint(11,22), 5)
      
      # find the nearest vertex within tolerance
      lstV = locator.verticesInTolerance(QgsPoint(33,44), 10)
      lstE = locator.edgesInTolerance(QgsPoint(33,44), 10)
      
      # find out in which polygons the point is located
      for m in locator.pointInPolygon(QgsPoint(33,44)):
        print "pt in polygon: ", m.featureId()


3. custom point locator - useful for analytic tools working without map canvas::

      locator = QgsPointLocator(layer)
      
      m = locator.nearestVertex(QgsPoint(1,1))

4. custom snapping utils - useful for analytic tools working without map canvas::

      utils = QgsSnappingUtils()
      utils.setMapSettings(settings)
      utils.setSnapToMapMode(QgsSnappingUtils.SnapAdvanced)
      cfg1 = QgsSnappingUtils.LayerConfig(layer1, QgsPointLocator.Vertex, 0.1, QgsTolerance.MapUnits)
      cfg2 = QgsSnappingUtils.LayerConfig(layer2, QgsPointLocator.Edge, 0.2, QgsTolerance.MapUnits)
      utils.setLayers([cfg1, cfg2])
      
      m = utils.snapToMap(QgsPoint(11,22))


GUI Changes
-----------

The snapping settings dialog will be updated to support the new mode "snap to current layer".
The new mode will be the default.
The user will be able to choose snapping type and tolerance.
The existing snapping settings (with a table listing individual layers) will be marked as "advanced" mode.



Performance Implications
------------------------

It is expected that snapping performance will be sped up significantly.
From some quick tests, the current snapping took ~30ms to find the closest vertex, while with new implementation
needed only less than 1ms. This is because current snapping needs to hit data provider (if the layer is not in editing mode).

There is some cost in initial indexing in ``QgsPointLocator``. For a layer with ~50K points this took about 100ms.
This is just one-time cost to build the index from data provider's features when queries are first needed.
No extra cost when map is rendered.


Test Coverage
-------------

New classes are designed in a way that they can be used in automatic tests easily. Core classes will have unit tests.


Further Considerations
----------------------

Some notes for the possible future improvements:

- it would be nice to support out of the box also ``QgsVectorLayerCache`` or any other object that can provide features

- it may be useful to build the index data structures in a background thread so the main thread is not blocked

- identify tool could make use of snapping utils for quick identification of features (on mouse hover)

- it may be useful to have indexing limited to a particular extent for very large layers



Backwards Compatibility
-----------------------

The existing classes used for snapping (``QgsSnapper``, ``QgsMapCanvasSnapper``) are left unmodified.
The new class ``QgsSnappingUtils`` is able to read the snapping configuration as used in existing project files.


Voting History
--------------

(required)
