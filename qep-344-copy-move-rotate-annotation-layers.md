# QGIS Enhancement Proposal (QEP): Add support for copy, move and rotate in annotation layers

**Date**: 2025-07-08
**Author**: Jean Felder ([@ptitjano](https://github.com/ptitjano))
**Contact**: jean dot felder at oslandia dot com
**Version**: QGIS 4.0

## Summary

This QEP proposes to enhance annotation layers in QGIS by adding support for moving, copying, and rotating annotation items. These improvements aim to align the annotating editing experience to that of the layout designer, improving overall UX consistency across the application.

## Proposed Solution

Currently, moving an annotation requires three separate clicks: one to select, one to initiate the move, and one to place the item. Moreover, multi-selection is not currently supported.

To improve usability and align with the layout designer experience, the current "Modify Annotation" tool will be split into two separate tools:

1. **Move/Copy/Rotate Annotation Items**
2. **Edit Annotation Geometry Vertices** (polygon, line and text-along-line annotations)

This separation allows for more intuitive interactions and clearer UX behavior.

### 1. Edit Annotation Geometry Vertices (Polygons and Lines)

This tool will retain its current behavior. It will allow editing of annotation geometries and support single-item selection only.

### 2. Move/Copy/Rotate Annotation Items (Text and Markers)

Once an annotation item is selected, users will be able to:

- **Move** the item using a click & drag with the primary mouse button.
- **Copy/Paste** the current selection using the standard shortcuts: `ctrl+c` (copy), `ctrl+x` (cut), and `ctrl+v` (paste). When pasted, the new items will be positioned relative to the current mouse cursor — the top-left corner of the pasted annotations will align with the cursor location. The `ctrl+shift+v` shortcut will be used for the paste in place action.
- **Rotate** the item interactively using a spinbox widget, displayed as an overlay in the top-right corner of the map canvas.

Multiple annotation items will be selectable using a **rubberband selection**. All operations will apply to the selected group. `Shift` modifier will allow to add elements to the current selection,`ctrl` to remove elements, and `alt` to invert the selection. Combinations of modifiers for intersect/within selections will also be possible, similar to the select features tool behaviour.

### Code Changes Overview

- **Annotation items**: Update `QgsAnnotationItem` classes to add rotation support

- **Map tools**: `QgsMapToolModifyAnnotation` will be split into:
  - `QgsMapToolModifyAnnotationTransform` (for move/copy/rotate)
  - `QgsMapToolModifyAnnotationVertexEdit` (for vertex editing)

- **Multi-selection**: Use `QgsRubberBand` to support group selection of annotation items.

- Extend the `QgsAnnotationItem` serialization logic to store rotation values in the project file.

## Deliverables

- Improved annotation interaction tools
- copy, move and rotate annotation items
- Support for selection, group operations

## Risks

None identified.

## Performance Implications

None anticipated.

## Backwards Compatibility

N/A
