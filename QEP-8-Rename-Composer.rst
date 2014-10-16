.. _qep#[.#]:

========================================================
QGIS Enhancement #: Rename Compositions to Print Layouts
========================================================

:Date: 2014/10/17
:Author: Nyall Dawson
:Contact: nyall.dawson@gmail.com
:Last Edited: 2014/10/16
:Status:  Draft
:Version: QGIS 2.8 and 3.0

#. Summary
----------

"Compositions" in QGIS should be renamed to "layouts".


#. Rationale
------------

This change is being proposed for a number of reasons:

- The term "composition" is an awkward name for this feature and does not
accurately reflect what this feature does. "Layout" is a much more widely
used term in desktop publishing and consequently a rename would improve
UX for new users who are unfamiliar with QGIS terminology.

- Other desktop GIS software packages (ArcGIS, MapInfo) use the term
"layout" to refer to print designs. A rename to match this would aid 
the transition from these packages to QGIS for new users.

- During the 2.8 cycle work is planned on a reporting framework for QGIS.
This framework would build off the existing composition features. While
the term "print composition" works to a degree, the phrase "report
composition" sounds horrible. "Print layout" and "Report layout" are much
friendlier sounding names for these features.

#. Potential issues
-------------------

- Documentation would need to be updated

- Existing users would need to be made aware of the change in name

- Existing blogs/stack exchange/mailing list answers would be outdated

#. Implementation Details
-------------------------

- Rename user visible strings for 2.8:

    - "Composition" -> "Print Layout"
    - "Print Composer" -> either "Print Layout" or "Layout Designer",
depending on the context. Currently "Print Composer" is used inconsistently.
    - "Composer" -> "Layout Designer"
    - "Composer Manager" -> "Layout Manager"
   
- For QGIS 3.0 and the next api break, rename internal classes and methods:

    - QgsComposition -> QgsPrintLayout (inheriting from QgsLayout)
    - QgsComposerItem (and subclasses) -> QgsLayoutItem
    - QgsComposerView - > QgsLayoutView
    - QgsComposer -> QgsLayoutDesigner

#. Further Considerations/Improvements
--------------------------------------

Feedback is requested as to whether or not the qgs project file format
should also be updated to reflect this change for QGIS 3.0, which would
break backwards compatibility with earlier QGIS versions.

#. Documentation
----------------

Documentation would need to be updated to suit.

#. Issue Tracking ID(s)
-----------------------

This proposal was originally filed as issue #5042.

#. Voting History
-----------------

(required)
