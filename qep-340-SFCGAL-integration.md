# QGIS Enhancement: Integration of SFCGAL Library

**Date** 2024/10/17
**Author** Loïc Bartoletti ([@lbartoletti](https://github.com/lbartoletti)), Benoit De Mezzo ([@benoitdm-oslandia](https://github.com/benoitdm-oslandia)), Jean Felder ([@ptitjano](https://github.com/ptitjano)), Julien Cabieces ([@troopa81](https://github.com/troopa81))
**Contact** loic dot bartoletti at oslandia dot com, benoit dot de dot mezzo at oslandia dot com, jean dot felder at oslandia dot com, julien dot cabieces at oslandia dot com
**Version** QGIS 3.XX

# Summary

This enhancement proposal outlines the integration of [SFCGAL](https://sfcgal.org) (Simple Features for Computational Geometry Algorithms Library) into QGIS. SFCGAL is an open-source library that provides advanced 3D geometry capabilities and is already well-integrated with PostGIS and GDAL.

This integration aims to enhance QGIS's 3D geometry (and 2D advanced) processing capabilities, particularly for applications in geology and urban planning.

## Background and Motivation

SFCGAL offers several compelling advantages for integration into QGIS:

1. It is the only open-source and OSGeo-compliant library capable of handling 3D geometry operations.

1. It handles, by default, geometry data as fractions of integers instead of decimal numbers to gain performance and precision

1. It has a proven track record of use cases, particularly in geology and urban planning.

1. It is already well-integrated with PostGIS and GDAL, two key components in the QGIS ecosystem.

1. It is well-packaged and supported across major distributions (Windows, Mac, Linux, *BSD, etc.).

1. It has minimal dependencies, with CGAL (header-only) and Boost (already required by other libraries in QGIS).

1. The integration is expected to have a low impact on the existing QGIS codebase and can be implemented conditionally, allowing users to enable or disable it as needed.

1. It enables the addition of new processing algorithms, expressions and advanced 2D functionalities not available in GEOS. See the proof-of-concept work in the [QSFCGAL plugin](https://plugins.qgis.org/plugins/qsfcgal/).

1. It brings the possibility to have 3D analyst toolbox as other 3D softwares

1. It supports 3D calculations and adheres to OGC standards for TIN and PolyhedralSurface (including SOLID: BrepSolid). Today, we have to implement hacks for some [operations](https://github.com/qgis/QGIS/pull/59031#discussion_r1793811622) and some are not possible to reimplement in QGIS. This is one of the reasons why SFCGAL is used in PostGIS and GDAL.

1. [Unlike GEOS](https://github.com/qgis/QGIS/pull/58959) [which drops Z/M](https://github.com/qgis/QGIS/issues/56158) data, SFCGAL can handle Z calculations (distance, 3D algorithms, etc.) and potentially M and nD. For example, in the Elevation Profile tool, it is sometimes necessary [to hack the calculation](https://github.com/qgis/QGIS/pull/54964#discussion_r1502939991) of intersecting geometries due to the removal of Z/M data.

1. Also in the elevation profile tool (for example, with geological data), the cut through intersections of several 3D volumes is not available with GEOS but is available in SFCGAL.

1. [A last example workflow](https://geotribu-fr.translate.goog/articles/2024/2024-08-22_de-la-tolerance-en-sig-geometrie-06-sfcgal/?_x_tr_sl=fr&_x_tr_tl=en&_x_tr_hl=en-US&_x_tr_pto=wapp#): operations may fail with GEOS (by using a decimal geometry data) but will succeed with SFCGAL (by using a fration kernel and preserving precision).

## Proposed Solution

The integration of SFCGAL into QGIS will involve the following key components:

* CMake flag to enable/disable SFCGAL compilation
* stateless class `QgsSfcgalEngine`
* geometry cached class `QgsSfcgalGeometry`

### CMake

We plan to add a CMake option `WITH_SFCGAL` to allow conditional compilation of SFCGAL support.

### QgsSfcgalEngine

We propose to create a new low-level stateless class `QgsSfcgalEngine`. This class will bind the SFCGAL C library structures and functions in the QGIS code base. It will also provide memory and error management tools.

This class will not be available in Python.

### QgsSfcgalGeometry

We propose to create a new high-level class `QgsSfcgalGeometry` to expose SFCGAL functions to QGIS in a similar way to `QgsGeos`.

This class will produce geometry objects in the SFCGAL format and also functions to convert them back and forth to `QgsAbstractGeometry`.

By keeping the SFCGAL geometry object during the whole object life, we can chain SFCGAL operations bypassing the need to transform geometries from/to SFCGAL/QGIS and thus we can continue to perform operation using the CGAL fraction computation kernel.

This key feature will enable more efficient processing in terms of time and accuracy.

For example we can chain calls to `QgsSfcgalGeometry` without losing precision:

```cpp
QgsSfcalGeometry poly2D_A(myWKTStringA);
QgsSfcalGeometry poly2D_B(myWKTStringB);
QgsSfcalGeometry poly3D_A = poly2DA.extrude(5);

QgsSfcalGeometry intersection = poly3D_A.intersection( poly2D_B.extrude( 5 ) );
```

As it was previously discussed, we don't plan to make `QgsSfcgalGeometry` inherit from `QgsGeometryEngine` because the different set of algorithm offered by GEOS and SFCGAL would imply a different API, and we see no advantages in having a common interface.

This class will be available in Python.

### Integration into QgsGeometry

At first, there is no plan to use SFCGAL algorithm within `QgsGeometry`. But some geometry functionalities in `QgsGeometry` are limited with GEOS due to Z/M drop or unmanaged kind of geometries (TIN, SOLID, etc.). Theses functionalities could be improved by using SFCGAL. To do so we will need to change each implied function and add an if/else condition which calls a new `isSFCGALCompatible` function. This is a similar approach to what GDAL does ([see also](https://github.com/OSGeo/gdal/blob/ce93839b1752fb40af064d5da49c85a0d7b8fc78/ogr/ogrgeometry.cpp#L8360)).

The `isSFCGALCompatible` function will, for example:

* let the priority to GEOS
* allow SFCGAL usage for PolyhedralSurface or TIN
* read some local setting to force the use of GEOS or SFCGAL

For example for the `intersection` function:

```cpp
QgsGeometry QgsGeometry::intersection( const QgsGeometry &geometry, const QgsGeometryParameters &parameters ) const
{
  if ( !d->geometry || geometry.isNull() )
  {
    return QgsGeometry();
  }

  mLastError.clear();
#ifdef WITH_SFCGAL
  if ( isSFCGALCompatible() || geometry.isSFCGALCompatible() )
  {
    QgsSfcgalGeometry sfcgalGeom( *d->geometry.get() );
    std::unique_ptr< QgsSfcgalGeometry> result( sfcgalGeom.intersection( *geometry.d->geometry.get(), &mLastError ) );

    if ( mLastError.isEmpty() )
      return QgsGeometry( std::move( result->asQgisGeometry( &mLastError ) ) );
    return QgsGeometry();
  }
  else
#endif
  {
    QgsGeos geos( d->geometry.get() );

    std::unique_ptr< QgsAbstractGeometry > resultGeom( geos.intersection( geometry.d->geometry.get(), &mLastError, parameters ) );

    if ( !resultGeom )
    {
      QgsGeometry geom;
      geom.mLastError = mLastError;
      return geom;
    }

    return QgsGeometry( std::move( resultGeom ) );
  }
}
```

## Use cases that leverage SFCGAL's capabilities

### Implement new processing algorithms and new expressions

#### Cross section extraction

The 3D intersection algorithm can be used in the elevation tool to extract the cross section data even when the capture curve has multiple segments.

#### Medial axis 2D

SFCGAL can calculate the centre line in longitudinal direction for 2D polygon features (e.g. road axis):

![](https://pad.oslandia.net/uploads/fbca3d1b-7fe3-4c6a-923b-f7045ba8843f.png)

#### Straight skeleton extrusion

One features in SFCGAL is the ability to generate “false” roofs by extruding the straight skeleton of a polygon.
Methods to convert 2D building footprints into 3D existed previously, but the quality and efficiency of the algorithm provided by CGAL allow for a significantly more effective solution for this use case, ensuring a precise and functional roof design.

![](https://pad.oslandia.net/uploads/2a6d0962-8a32-4dd8-97d4-33b3e055c67d.png)

Here is a example to do this operation:

```cpp
QgsVectorLayer outputLayer;
for (auto feat: features) {
    QgsSfcgalGeometry building (*feat.geometry().constGet());
    QgsSfcgalGeometry polySurf = building.extrudePolygonStraightSkeleton(feat.attribute("build_height").toDouble(), feat.attribute("roof_height").toDouble());

    QgsFeature outFeat;
    outFeat.setGeometry(QgsGeometry(polySurf.asQgisGeometry()));
    outputLayer.addFeature(outFeat);
}
```

#### Visibility algorithms

These algorithms enhance the capability to analyze visibility between geometric objects; a crucial feature in a wide array of applications, from urban planning to robotics.

These algorithms enable determining visible areas from a point or an edge, as illustrated in the following example.

![](https://pad.oslandia.net/uploads/9179a140-2c33-4e44-bb67-48b1e35186b3.png)

## Deliverables

1. Implementation of `QgsSfcgalEngine` and `QgsSfcgalGeometry` classes.

2. CMake configuration for conditional SFCGAL support.

3. A set of new processing algorithms and expressions that utilize SFCGAL functionality.

4. Documentation for developers and users on how to use SFCGAL features in QGIS.

5. Unit tests to ensure proper functioning of the SFCGAL integration.

### Affected Files

* `src/core/geometry/qgsgeometryengine.h`

* `src/core/geometry/qgsabstractgeometry.h`

* `src/core/geometry/qgssfcgalengine.h` (new file)

* `src/core/geometry/qgssfcgalengine.cpp` (new file)

* `src/core/geometry/qgssfcgalgeometry.h` (new file)

* `src/core/geometry/qgssfcgalgeometry.cpp` (new file)

* `CMakeLists.txt`

* `cmake/findSCFGAL.txt`

* Various files in the `src/analysis` directory for new processing algorithms

* Various files in the `src/core/expression` directory for new expressions

## Risks

1. Potential increase in build complexity due to the additional dependency.

2. Low: Maintenance burden of keeping SFCGAL integration up-to-date with future QGIS versions. Like other dependencies.

## Performance Implications

The performance impact is expected to be minimal for users who do not enable SFCGAL functionality. For those who do use SFCGAL features, there may be a slight increase in memory usage and processing time for complex 3D operations. However, this is offset by the significant capabilities gained in 3D geometry processing.

## Further Considerations/Improvements

1. Explore the possibility of extending QGIS's 3D capabilities to leverage SFCGAL's 3D geometry support.

2. Consider developing a dedicated SFCGAL toolbox in the QGIS processing framework.

3. Investigate potential synergies between SFCGAL and other QGIS components.

## Backwards Compatibility

The proposed implementation should not affect existing QGIS functionality. All SFCGAL-related features will be optional and can be disabled at compile-time or runtime. Existing GEOS-based geometry operations will remain unchanged.

## Issue Tracking ID(s)

(To be assigned upon acceptance of this proposal)

Funded by: CEA/DAM [@renardf](https://github.com/renardf), CP4SC/France Relance/European Union
