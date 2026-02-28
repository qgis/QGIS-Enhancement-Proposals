# QGIS Enhancement: On-the-fly contour line labeling for raster layers

**Date** 2026/03/01

**Author** James McGuiness (@Geojim)

**Contact** via GitHub

**Version** QGIS 3.44

# Summary

Add on-the-fly contour line labels for raster layers rendered with the Contour renderer. A second pass of GDAL's contour API generates label geometries independently from the renderer, and labels are placed along them via the existing PAL labeling engine. Accessed through the Labels tab in Layer Properties, consistent with existing vector labeling. See [issue #53415](https://github.com/qgis/QGIS/issues/53415).

## Proposed Solution

### Architecture

Two new classes:

- **`QgsRasterContourLabelProvider`** (internal, not exposed to Python) — reads the visible raster extent, generates contour lines via `GDAL_CG_Create`/`GDAL_CG_FeedLine`, transforms to map CRS, and registers each line with the PAL engine using `Qgis::LabelPlacement::Line` with `MergeConnectedLines`.

- **`QgsRasterLayerContourLabeling`** (public API) — configuration class storing text format, numeric format, label-index-only flag, priority, z-index, placement, thinning, and scale visibility settings.

### Integration with Symbology

Relevant contour parameters (band, interval, index interval, downscale) are read from `QgsRasterContourRenderer` at render time.

### UI

The Labels tab gets a "Contour Labels" option in the dropdown. The widget shows:

- Number format (for elevation values)
- "Label index contours only" checkbox
- Standard text formatting, placement, rendering, and thinning controls

![Labels widget](https://github.com/user-attachments/assets/2866c03e-18a2-41f1-b5d0-9426e2153e48)

### Affected Files

New:
- `src/core/raster/qgsrastercontourlabeling.cpp/.h`
- `src/gui/raster/qgsrastercontourlabelsettingswidget.cpp/.h`

Modified:
- `src/core/raster/qgsrasterlabeling.cpp/.h` — factory dispatch, make `QgsRasterLayerLabelProvider` extensible
- `src/core/CMakeLists.txt`, `src/gui/CMakeLists.txt`
- `src/gui/raster/qgsrasterlabelingwidget.cpp` — add dropdown entry

## Deliverables

- Core contour label generation and configuration classes
- GUI widget integrated into existing labeling framework
- SIP bindings and Python unit tests
- Project XML save/load persistence

A working implementation is available at [qgis/QGIS#65140](https://github.com/qgis/QGIS/pull/65140).

## Risks

Minimal. The downscale factor keeps contour generation fast even for large rasters, and PAL's existing overlap handling and thinning settings prevent label clutter. Tested and no noticeable impact.

## Performance Implications

The label provider runs GDAL contour generation independently from the renderer, so there is a second hit on the contour generation for each render cycle. The overhead is proportional to the visible extent and downscale factor. In practice this is acceptable because the GDAL contour API is fast relative to the overall render pipeline, and the downscale factor (typically 4x) significantly reduces the input data size. If needed, the label pass could use a higher downscale factor than the renderer since labels only need approximate line positions for placement, not pixel-perfect geometry. No modifications to the PAL engine are required.

## Backwards Compatibility

New feature only. The change to `QgsRasterLayerLabelProvider` (extensible instead of final) is backwards compatible.

## Alternative Considered

A cache-based approach where the renderer shares its contour geometries with the label provider was considered but rejected. It would tightly couple the renderer and label provider, require managing cache lifetime across render cycles, and the renderer's block-based tiling fragments contour lines into disconnected segments that complicate label placement.

## Issue Tracking ID(s)

- https://github.com/qgis/QGIS/issues/53415
