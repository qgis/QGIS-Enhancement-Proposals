# QGIS Enhancement 322: Add support for point layer edition from the profile tool

**Date** 2025/02/13

**Author** Jean Felder (@ptitjano), Jacky Volpes (@Djedouas)

**Contact** jean dot felder at oslandia dot com, jacky dot volpes at oslandia dot com

**Version** QGIS 3.44

# Summary

This proposal aims to enable the editing of point layers directly from the elevation profile. It will be possible to add new points, move existing ones, and delete them. Additionally, selecting points from the elevation profile will be possible. Editing directly within the elevation profile will make it easier to precisely adjust the elevation of 3D points.

## Proposed Solution

The main challenge of this feature is ensuring that these new functionalities do not conflict with the existing editing options in the main canvas. To make the solution as clear as possible, we propose to:

- Add a new editing section to the toolbar within the elevation profile panel.
- Display the editing icon for the layer currently selected in the elevation profile tree view.
- Synchronize edition state between the main canvas and the elevation profile: enabling edition on a layer from the elevation profile will activate it in the main canvas, and vice versa.

Tolerance will not be handled when adding or moving points. This means that the points will always be placed on the profile line.  
This editing toolbox will not be enabled if the selected layer is not a point layer.

The different operations (Add, Delete, Move, Select) will be implemented by creating new tools that inherit from `QgsPlotTool`.

This branch already contains a rough implementation: https://github.com/ptitjano/QGIS/tree/capture-profile-points

## Deliverables

- A new editing toolbox in the profile tool
- Add / Move / Delete / Select points from the profile tool

### Affected Files

- src/app/elevation/qgselevationprofilewidget.{h,cpp}
- src/app/elevation/qgselevationprofiletooladdpoint.{h,cpp} (new files)
- src/app/elevation/qgselevationprofiletoolmovepoint.{h,cpp} (new files)
- src/app/elevation/qgselevationprofiletoolselectfeatures.{h,cpp} (new files)
- src/core/elevation/qgsprofilerenderer.cpp
- src/core/layout/qgslayoutitemelevationprofile.cpp
- src/gui/elevation/qgselevationprofilelayertreeview.cpp
- src/gui/elevation/qgselevationprofilecanvas.cpp
- src/gui/plot/qgsplotrubberband.{h,cpp} (new files)

## Risks

Increased complexity and maintenance burden in the profile tool code.

## Performance Implications

N/A

## Backwards Compatibility

Not Applicable.


Funded by: BRGM
