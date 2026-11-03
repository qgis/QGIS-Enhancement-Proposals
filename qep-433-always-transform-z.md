# QGIS Enhancement 433: Always transform Z values when doing transforms (API break)

**Date** 2026/11/03

**Author** Dominik Cindric (@ViperMiniQ)

**Contact** viper dot miniq at gmail dot com

**Version** QGIS 4.6

# Summary

Geometry transforms currently ignore Z by default. **QgsAbstractGeometry::transform** sets **transformZ = false** as default.
If source and destination CRSes have different height references, X and Y are transformed, but Z is just copied over from the source. As a result, geometries contain wrong heights in destination CRS.

This QEP proposes breaking the API changing the default to **transformZ=true** in the method signature.

The same effect this QEP would have would also be possible if we just update all calls to the method, but we feel skipping the Z transform should be an exception, and not the default behavior.

## Proposed Solution

Change default transformZ param from false to true in qgsabstractgeometry.h and qgsgeometry.h and all subclasses.

Switch the **TRANSFORM_Z** from default false to true in **Reproject Layer** processing algorithm.

Remove all reduntant passes of the true flag when calling the appropriate **transform()** method.

## Affected 

Any performed transforms now take into account the Z value, instead of passing it through algorithms unaffected.

What changes in the output, by source and destination CRS (measured with **Reproject Layer**):

**transformZ = false**

| | to: 2D CRS | to: 3D CRS | to: geocentric CRS |
| --- | --- | --- | --- |
| **from: 2D CRS, 2D geom** | no Z - correct | no Z - correct | no Z - correct |
| **from: 2D CRS, 3D geom** | Z copied - correct | Z copied - correct | X/Y computed at height 0, Z copied - **wrong** ❌ |
| **from: 3D CRS, 2D geom** | no Z - correct | no Z - correct | no Z - correct |
| **from: 3D CRS, 3D geom** | Z copied - correct | Z copied - **wrong** ❌| X/Y computed at height 0, Z copied - **wrong** ❌ |
| **from: geocentric CRS** | Z copied - **wrong** ❌ |  Z copied - **wrong** ❌ | no change |

**transformZ = true**

| | to: 2D CRS | to: 3D CRS | to: geocentric CRS |
| --- | --- | --- | --- |
| **from: 2D CRS, 2D geom** | no change | no change | no change |
| **from: 2D CRS, 3D geom** | no change | no change | **changed** - Z read as ellipsoidal height ⚠️ |
| **from: 3D CRS, 2D geom** | no change | no change | no change |
| **from: 3D CRS, 3D geom** | no change | **changed** - Z converted to the destination vertical datum - correct ✅ | **changed** - X/Y/Z computed from the real height - correct ✅ |
| **from: geocentric CRS** | **changed** - ellipsoidal height in Z - correct ✅ | **changed** - Z in the destination vertical datum - correct ✅ | no change |

### Affected Files

**qgsabstractgeometry.h** (and **all subclasses**)

**qgsgeometry.h**

*any files explicitly calling **transform()** with **transformZ=true** will be updated to not pass the unneeded argument.

## Risks

A potential to get unexpected (but correct) values for the Z component.

Mesh editing has been identified as being broken by this, and an explicit **transformZ=false** will be passed in order to keep the behavior of the algorithm.
Same thing can be applied to other places if found at that they are broken by this - as a quick fix, better solution would be to fix the underlying issue.

PyQGIS code calling **transform()** on geometries gets different Z values.

**processing.run('native:reprojectlayer', …)** that don't change the **TRANSFORM_Z** flag get a different output for Z. This optional parameter was added a bit less than a year ago (merged 21th Nov 2025: https://github.com/qgis/QGIS/pull/63936), after the latests LTR (3.44).

There is a possibility of affecting 3rd parties - plugin developers using geometry transforms and the workflows utilizing the **native:reprojectlayer** that depend on the Z values not changing (which, hopefully, shouldn't be many, if any at all).