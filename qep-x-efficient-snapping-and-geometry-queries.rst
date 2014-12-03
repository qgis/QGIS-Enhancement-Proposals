.. _qep#[.#]:

========================================================================
QGIS Enhancement #: Efficient Snapping and Geometry Queries
========================================================================

:Date: 2014/12/03
:Author: Martin Dobias
:Contact: wonder.sk at gmail dot com
:Last Edited: 2014/12/03
:Status:  Draft
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

The current snapping implementation is spread across several classes (QgsGeometry, QgsVectorLayer, QgsSnapper,
QgsMapCanvasSnapper) and includes some extra features (like "excluded points" or mode for "topological editing")
as a core functionality.

Ideally the implementation should focus just on search for the closest vertices/segments
and leave extra features for higher level - to keep the querying core simple and make it easier to optimize it.

The aim is to simplify the implementation to the following structure:

1. low-level querying - for a set of geometries (typically one vector layer), find out the nearest
   vertex/segment or all near vertices/segments within tolerance.
2. high-level querying - operates on top of several layers, combines results from lower level
3. extra functionality - routines that are not directly related to querying, but are useful in context of snapping
   and are provided for user's convenience

Currently the snapping uses either simple geometry cache or iterates over layers if cache is not available.
The cache however does not have an efficient data structure for spatial querying (data are stored
in a map with feature IDs as keys and geometries as values). Introduction of R-trees should improve
the performance of queries even with greater amount of geometries and greater complexity of those (from
linear search to logarithmic). This should enable more interactive GUI functionality later,
such as mouse over effects for various tools or continuous mode for identify tool
(like Value Tool, not having to click on each feature).

