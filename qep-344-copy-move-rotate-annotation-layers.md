# QGIS Enhancement Proposal (QEP): Add support for copy, move and rotate in annotation layers

**Date**: 2025-07-08
**Authors**: Jean Felder ([@ptitjano](https://github.com/ptitjano)) / Mathieu Pellerin ([@nirvn](https://github.com/nirvn))
**Contact**: jean dot felder at oslandia dot com / mathieu dot opengis dot ch
**Version**: QGIS 4.0

## Summary

This QEP proposes to enhance annotation layers in QGIS by adding support for moving, copying, and rotating annotation items. These improvements aim to align the annotating editing experience to that of the layout designer, improving overall UX consistency across the application.

## Proposed Solution

Currently, moving an annotation requires three separate clicks: one to select, one to initiate the move, and one to place the item. Moreover, multi-selection is not currently supported.

To improve usability and align with the layout designer experience, a new “Select/Move Annotations” tool will be added alongside the preexisting "Modify Annotation", which will be renamed to “Edit Annotation Nodes” to harmonize these with what is present in the layout designer.

This separation will allow for more intuitive interactions and clearer UX behavior for both tool while closely matching the layout designer UX.

### 1. Edit Annotation Nodes

This renamed tool will retain its current behavior. It will allow editing of annotation nodes (geometry vertices and additional configuration nodes such as callout endpoint) and support single-item selection only.

The tool will only allow single selection. When clicking on a node with a preexisting multi-selection state, the tool will change the selection to only retain the one annotation tied to the clicked node. The tool will also retain it’s click-click behavior. While it means diverging from the layout designer, it matches other tools tied to map canvases such as the node editor, the georeferencer, etc. 

### 2. Select/Move Annotations

This tool will rely on the preexisting QgsGraphicsViewMouseHandles class used by the layout designer to offer the following functionalities against a single or multiple selected items:

- **Move** the item(s) using a click & drag with the primary mouse button.
- **Resize** the item(s) interactively using selection corner resizing handles.
- **Rotate** the item(s) interactively using the selection corner rotation handles.
- **Copy/Paste** the item(s) using the standard shortcuts: `ctrl+c` (copy), `ctrl+x` (cut), and `ctrl+v` (paste). When pasted, the new items will be positioned relative to the current mouse cursor — the top-left corner of the pasted annotations will align with the cursor location. The `ctrl+shift+v` shortcut will be used for the paste in place action.

The selection behavior will match the layout designer through the shared base class. The `shift` modifier will allow to add elements to the current selection,`ctrl` to remove elements, and `alt` to invert the selection. Combinations of modifiers for intersect/within selections will also be possible, similar to the select features tool behavior.

Single selection should insure the style panel is updated to allow for customization of the annotation. When multiple items are selected, the styling panel should update to indicate the editing is not possible due to multiple items being selected.

### Code Changes Overview

- **Annotation items**: Update `QgsAnnotationItem` to move preexisting rotation properties from derived classes into this base class.

- **Map tool**: `QgsMapToolModifyAnnotation` will retain its role focusing on node editing with an update to its iconography and label, while a `QgsMapToolSelectAnnotation` tool will be added to handle selection, movement, resizing, rotation, and copy/pasting. The new tool will rely on QgsGraphicsViewMouseHandles to provide the necessary GUI to handle resizing, rotation as well as rendering the selection rectangle(s).

## Deliverables

The implementation will deliver an improved user experience when interacting with annotation items by adopting a new tool that will match the layout designer select/move tool behavior. This will allow for users to conduct movement, resizing, and rotation operations against a selection containing one or more items at the same time. 

Copy/pasting of one or more items will also be made available through that new tool, as users would within the layout designers when selecting items.

## Risks

None identified.

## Performance Implications

None anticipated.

## Backwards Compatibility

N/A
