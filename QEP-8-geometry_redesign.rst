.. _qep#[.#]:

========================================================================
QGIS Enhancement 8: Geometry redesign
========================================================================

:Date: 2014/10/30
:Author: Marco Hugentobler
:Contact: marco at sourcepole.ch
:Last Edited: 2014/10/30
:Status:  Draft
:Version: QGIS 2.7
:Sponsor: Canton of Solothurn, Switzerland

.. note::

    See :ref:`QEP 1 <qep1>` for description of QEP process.

1. Summary
----------

The mmsql geometry standard describes some features not supported by the current geometry class:

- Curved geometries (circular arcs
- Compound types (compoundcurve, curvepolygon, geometrycollection)
- Z-coordinates
- M-coordinates (e.g. used for linear referencing)

This QEP proposes a redesign of the geometry system to support these features. As a positive side effect, the code should be more flexible and easier maintainable (e.g. possibility to add new geometry types likes splines).

2. Possible Applications
------------------------

The following examples come to my mind (surely there are more):

1. In Switzerland, circular arcs and compound types (compoundcurve, curvepolygon) are important in public data like cadastral surveying data . These types are already supported in PostGIS, and work is underway to support it in OGR as well. In order to be able to see and edit this data, it is a logic step to support it in QGIS too.

2. It is common to have 3D data in PostGIS. Currently, depending on the edit method used, it can happen that QGIS converts a geometry to 2D during editing. QGIS should at least preserve the existing z-Values if possible (e.g. if a vertex is moved to a new position). To achieve that, it needs an internal geometry representation that supports z-values. Maybe we will see more 3D plugins if storing z-coordinates is supported by the core library.

3. Linear referencing is an important feature to avoid redundancy for geographic data. Currently, it is done either with views on DB side or, for shapefiles, interpreting z-values as m and then doing the computations in the analysis library (using event layer plugin as GUI). If the geometry model supports m-values, linear referencing can be solved properly and for all datasources which support m-values.

3. Proposed solution
------------------------

It is proposed to build a class hierarchy according to the mmsql iso standard with QgsAbstractGeometryV2 as the base class. Working with inheritance, it will be straightforward to support nested types, like compoundcurve or geometry collecion. As the current geometry class is used in so many places in QGIS, it is proposed to keep the interface QgsGeometry and direct the calls to it to a new geometry object. This means that existing code should just continue working while offering the new possibilities.

For editing, it is planned to adapt the 'add feature' tool with the possibility to digitise circular arcs and to adapt the node-tool with the  possibility to modify start/end/midpoints of circular arcs.

For geometry analysis and some edit methods, QGIS uses the geos library. As geos does not support circular arcs, curved geometries are segmentised if using one of those methods.

4. Class design
--------------------------

.. image:: uml.png 

The UML diagram of the proposed solution contains the following elements:

- QgsGeometry has the same API as the current class. Internally, it holds a pointer to the new geometry class and redirects calls to it. To avoid performance overhead, QgsGeometry uses implicitely sharing. 
- QgsAbstractGeometryV2 is the base class of the new geometry classes. Instantiable classes are QgsPointV2, QgsLineStringV2, QgsCircularStringV2,QgsCompoundCurveV2, QgsCurvePolygonV2, QgsPolygonV2, QgsGeometryCollectionV2, QgsMultiPointV2, QgsMultiSurfaceV2, QgsMultiCurveV2
- QgsGeometryEngine / QgsGeos handles analytical operations like buffer/union/intersection, etc. as well as import/export between geometry classes and geos. All this code is separated from the geometry classes now. The API has been further extened to prepare a geometry for better performance in case of repeaded topological operations.
- In QgsAbstractGeometryV2 and subclasses, methods like 'draw()', 'transform()', 'mapToPixel()' and (in future) 'vertices()' are provided. The idea is that code (e.g. for rendering) calls these methods instead of having a switch over the geometry types and pulling out the vertices. Old code (or code for very special rendering operations) can still call asPolygon(), asPolyline() etc. and receives a segmentized geometry in case of curves. 

5. QgsAbstractGeometryV2 interface
----------------------------------

- Geometry access: code should, wherever possible, call methods of QgsAbstractGeometryV2 (e.g. call QgsAbstractGeometryV2::transform() instead of retrieving the coordinates and transform them). In cases the content of the geometry really needs to be retrieved, there are the following access methods:

  - QgsPointV2::x(), QgsPointV2::y(), QgsPointV2::z(), QgsPointV2::m() to get the coordinates of a point. 
  - QgsAbstractGeometryV2::is3D(), QgsAbstractGeometryV2::isMeasure() to test, if a geometry has z/m values.
  - QgsAbstractGeometryV2::coordinateSequence( QList< QList< QList< QgsPointV2 > > >& coord ) retrieves the vertices for all geometry types
  - QgsCurveV2::points( QList<QgsPointV2>& pt ) retrieves vertices of curves.
  - QgsCurvePolygonV2::exteriorRing() and QgsCurvePolygonV2::exteriorRing() gets the rings for polygons
  - QgsGeometryCollectionV2::geometryN( int i ) accesses the parts of multigeometries
  
- Geometry construction

  - QgsPointV2( double x, double y, double m, double z )
  - QgsLineStringV2::setPoints( const QList<QgsPointV2>& points ), QgsCircularStringV2::setPoints( const QList<QgsPointV2>& points )
  - QgsCompoundCurveV2::addCurve( QgsCurveV2* c )
  - QgsCurvePolygonV2::setExteriorRing( QgsCurveV2* ring ), QgsCurvePolygonV2::setInteriorRings( QList<QgsCurveV2*> rings )
  - QgsGeometryCollectionV2::addGeometry( QgsAbstractGeometryV2* g )

6. Performance implications
----------------------------

First tests indicate that the rendering performance can be similar to the current state (2.6). It is planned to do some testing with qgis_bench to avoid significant performance regressions. It will however be impossible to test all possible rendering option.

7. Test implementation
----------------------------

A test implementation is in the branch https://github.com/mhugent/Quantum-GIS/tree/geometry_mmsql . At the moment, it is incomplete, not fully tested and editing of geometries is completely disabled. It should however already give a hint about the direction of development.

8. Questions and Answers
----------------------------

Q: Will the segmentising be done automatically so that plugins which also work on geometries but not with geos can also use this solution?

A: Yes, it will be done automatically with QgsGeometry::intersection(), QgsGeometry::asPolyline, etc. The only case where it is not automatically done is if the plugin calls asWkb() and parses the binary itself. In that case, the plugin still has the possibility to convert the geometry to a linear one (QgsGeometry::convertToStraightSegments)

Q: Looking at your branch I don't see any unit testing implemented for this work. Is this something which is planned? I strongly believe that something as fundamental as the geometry classes must be accompanied by a comprehensive test suite. 

A: There are some tests in testqgsgeometryimport.cpp. It is indeed planned
to add much, much more tests once the architecture is stable.

Q: Why not reusing the existing classes to make the geometry hierarchy?

A: More flexibility and backward compatibility

Q: 'QgsGeometryEngine' is a more descriptive name than 'QgsVectorTopolgy'

A: Agreed, changed the class name

Q: The methods which are delegated to geos do not have to be in the geometry class interface

A: Agreed. The code is changed such that QgsAbtractGeometryV2 does not have a relation to QgsGeometryEngine

Q: there is a naming clash between QgsPoint and QgsPointV2 - they are meant for completely different purpose, yet QgsPointV2 looks like "improved" QgsPoint which is confusing. Especially, what will happen if we get rid of old QgsGeometry and the V2 suffix will be dropped?

A: The current QgsPoint will be removed and QgsPointV2 becomes QgsPoint

Q: Will the new geometry classes have implicitely sharing?

A: It is not planned. However it can be added to the new classes (needs to be done separately for each one)

Q: May I suggest to use iterator pattern for various methods that currently return (possily nested) containers? I mean methods like
QgsAbstractGeometryV2::coordinateSequence()

A: Added QgsAbstractGeometryV2::nextVertex to iterate over the vertices of a geometry

Q: Regarding the geometry access from QgsFeature - are the geometries still going to be accessed via f.geometry() or will there be a new method to get access directly to QgsAbstractGeometryV2 ?

A: Added QgsFeature.geometryV2()

Q: would it be possible to have a code examples in Python demonstrating the new geometry API? e.g. code that reads features and dumps geometry contents

A: Here are some examples (hope to add more in future):

QgsFeature::geometryV2(). So to dump the geometry content (in python), use code like this:

print feature.geometryV2().asWkt()

To create geometries in Python, do e.g. (for a linestring)

lineCoords = [QgsPointV2(100,100), QgsPointV2( 110, 100 ), QgsPointV2( 110, 110 )]
lineString = QgsLineStringV2()
lineString.setPoints( lineCoords )

A zm-Geometry can be created by having zm points in the coordinate list:

lineCoords = []
lineCoords.append( QgsPointV2( QgsWKBTypes.LineStringZM, 100, 100, 23, 1.0 ) )
lineCoords.append( QgsPointV2( QgsWKBTypes.LineStringZM, 110, 100, 33, 2.0 ) )
lineCoords.append( QgsPointV2( QgsWKBTypes.LineStringZM, 110, 110, 43, 3.0 ) )
lineString = QgsLineStringV2()
lineString.setPoints(lineCoords)
