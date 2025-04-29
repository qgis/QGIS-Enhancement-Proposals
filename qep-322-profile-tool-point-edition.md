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
- Synchronize selection: it is the same between the elevation plots and the main layer.

The different operations (Add, Delete, Move, Select) will be implemented by creating new tools that inherit from `QgsPlotTool`. 

This editing toolbox will not be enabled if the selected layer is not a point layer.

### Selection

Select features by clicking on them or with a rectangle selection. keyboard modifiers will allow to select points more precisely:

 - Shift to add to current selection
 - Control to remove from current selection

### Add features

In editing mode, activate it by clicking the "Add Point Feature" button in the elevation profile panel. then click on the elevation profile to add a new point. If necessary, the attribute form will immediately open. Snapping will be handled.

### Move features

In editing mode, activate it by clicking the "Move Points Feature" button in the elevation profile panel. Select a point by clicking on it and then click on its new location. Moving points will have a special "lock abscissa" keyboard modifier to avoid changing x/y coordinate while moving. It will also handle snapping. 

### Delete features

Deletion of features will work on selection. First, select features by clicking on them or using a rectangle selection. Then delete them by clicking the "Delete Features" button from the elevation profile toolbar.

### Tolerance

 - added points will be placed on the profile line
 - points can be picked/selected in the tolerance zone to be moved or deleted but moved points are reprojected on the profile line (x/y are changed when z is moved)

## Deliverables

- A new editing toolbox in the profile tool
- Add / Move / Delete / Select points from the profile tool

This branch already contains a rough implementation: https://github.com/ptitjano/QGIS/tree/capture-profile-points

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
