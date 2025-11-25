# QGIS Enhancement: Multipart geometry labeling enhancements

**Date** 2025-11-18

**Author** Nyall Dawson (@nyalldawson)

**Contact** nyall dot dawson at gmail dot com

**Version** QGIS 4.0

# Summary

Currently, when labeling multi-part geometries, QGIS has two behaviors:

1. Only the largest part is labeled. This is the default behavior.
2. Every part is labeled with the same text. This behavior is actived when the user checks the
   "Label every part of multi-part features" checkbox.

This QEP proposes a new option for labeling multi-part geometries in order to give users more cartographic
control over the exact appearance and placement of their labels. The user interface will be reworked
to accommodate this new option.

## Proposed Solution

The existing checkbox for “Label every part of multi-part features” will be re-worked to use a combobox to expose the
choice of multi-part handling options.

The available options will be:

- “Label largest part only”. This corresponds to the current default behavior, i.e. when the user has left the
  "label every part" checkbox unchecked.
- “Label every part with same text”. This corresponds to the current behavior which is activated when the user manually
  checks the "label every part" checkbox
- “Place lines on parts”. This is a brand new behavior, as described below.

To accomodate the new behavior selection, an enum `Qgis::MultiPartLabelingBehavior` will be introduced. The existing
`QgsPalLayerSettings::labelPerPart` boolean will be deprecated, and replaced with a getter/setter for the new enum
in the existing `QgsLabelPlacementSettings` class. Python compatibility code will be introduced to ensure the stable
API for `labelPerPart` remains intact.

### Place lines on parts

This new behavior option will split the label text over the parts of a multi-geometry feature.

The label text will be split at new line characters, and each line will be placed separately over corresponding
parts from the input feature geometry.

If the multipart geometry does not contain sufficient parts for the label text then the excess lines will be ignored.

The example image below shows a two-part line string, with the "Victory\nBay" (note the newline character!) text
from the feature being split over the two parts.

![example.png](images/qep405/example.png)

## Risks

None

## Performance Implications

None, this is a new setting and the default behavior will remain unchanged.

## Backwards Compatibility

N/A - the new modes will only be available in QGIS 4.0+.
