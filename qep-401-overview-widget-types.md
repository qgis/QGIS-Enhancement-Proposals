# QGIS Enhancement: Overview of widget types, constraints and default values

**Date** 2025/01/07

**Author** Germán Carrillo (@gacarrillor), Mathieu Pellerin (@nirvn)

**Contact** german at opengis dot ch, mathieu at opengis dot ch

**Version** QGIS 3.4X.X+

# Summary


In the vector layer properties, QGIS allows users to drag available layer fields from a field panel and drop them in a form layout panel. Once there, each form layout item has a widget type assigned with a default configuration. These widget types are currently only shown individually in a third panel, where users can change both the widget type and its configuration.

![Screenshot from 2024-11-24 22-22-18](https://github.com/user-attachments/assets/47cee69d-6ff2-4f50-9069-eab44bb66757)

When a layer has a relatively large number of fields, there is a need to have an overview of widget types, rather than just seeing them one by one. This can be easily extended to other field settings like constraints and default values.

We propose to tackle this via visual hints in field and form item panels, as well as bringing handy stuff to improve user experience.

## Proposed Solution

Switch from current item-based tree widget to model-based tree view to ease maintenance and gain flexibility.

Use the form layout item’s decoration role (i.e., item’s icon) to show its type of widget. This is appropriate because all items will have one and only one type of widget.

Use right-aligned indicator icons (like those that are used in the QGIS layer tree for filters and missing data sources, among others) for constraints and default values. The indicators will only be shown when constraints and/or default values are present, and details about specific configurations will be displayed via tooltips.

This work includes the graphic design of almost 20 icons for widget types, as well as a couple of icons for indicators of constraints and default values.


#### Other UI/UX enhancements

 + **Filter/search box**, taking advantage of the model-based tree view.
 + **Form preview**, to give users the option to see what they'll get easily and in the right place, i.e., without having to follow additional steps like closing Layer properties, enabling editing and adding a feature just to see what they've just configured in action. The preview will be opened in a new window, parented to the Vector Layer Properties dialog.
 + In the form layout panel, users will have the possibility to **switch between aliases and item names**, depending on their needs. This is particularly handy to easily find form layout items as they will be displayed in the generated input form, which may be cumbersome nowadays.

## Deliverables

Pull request(s) in QGIS repo implementing the enhancements listed and described above, including the new set of icons.

### Affected Files

`src/gui/attributeformconfig/*`

`src/gui/vector/qgsattributesformproperties.cpp`

`src/gui/vector/qgsattributesformproperties.h`

...

## Risks

None.

## Performance Implications

Not applicable, since the proposal deals with displaying already available data from items.

## Backwards Compatibility

The proposal will also work with projects from older QGIS versions.

## Issue Tracking ID(s)

https://github.com/qgis/QGIS/issues/58973
