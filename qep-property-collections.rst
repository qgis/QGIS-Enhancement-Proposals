.. _qep#[.#]:

====================================================
QGIS Enhancement #: Inheritable property collections
====================================================

:Date: 2014/11/22
:Author: Nyall Dawson
:Contact: nyall dot dawson at gmail dot com
:Last Edited: 2014/11/22
:Status:  Draft
:Version: QGIS 2.8 or above

1. Summary
----------

This QEP concerns a system of managing properties for an object. Properties are grouped in
collections, and can be overridden in a specified order of precedence.

This proposal is being driven by both layout and text/labelling work. Specifically:

- the labelling engine has a need for predefined label styles. Label properties could be
  set globally, per project, via a predefined style, or overriden for a particular layer.

- layouts also have a requirement for overridable properties. Layout item settings could
  be set globally (eg, font size), per project (eg font family), via a "master template"
  and finally individually per layout item.

This proposal consists of both a framework for collecting and retrieving properties,
and a GUI widget for controlling them.

2. Proposed Technical Solution
------------------------------

- Create a class "QgsProperty". This class stores a QVariant for the property's value
  and a bool for whether the property is currently active or not

- A class "QgsPropertyCollection" would be created, which stores a QMap of a property's
  name to corresponding QgsProperty.

- A class "QgsOverridablePropertyCollection" (suggestions welcome for a better name!),
  which stores an ordered list of QgsPropertyCollections. QgsOverridablePropertyCollection
  will have a method for retrieving the value from the first QgsPropertyCollection
  for which a particular property is active.

- A widget "QgsOverridablePropertyButton" would be created. This would follow the same
  visual style as the existing QgsDataDefinedButton widget. The widget's drop down
  menu would show all available QgsPropertyCollections for which the property can be 
  inherited. For instance, for layouts, this would show "Disabled", "From master template",
  "From project" and "QGIS Default". For labelling, this would show "Disabled", "From style",
  "From project" and "QGIS Default". 

3. References
-------------

See some discussion leading up to this QEP at https://github.com/qgis/QGIS/pull/1550

4. Voting History
-----------------

(required)
