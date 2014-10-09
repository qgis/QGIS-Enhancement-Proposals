.. _qep6

========================================================================
QGIS Enhancement 6: advanced digitizing tools
========================================================================

:Date: 2014-10-15
:Author: Denis Rouzaud
:Contact: denis.rouzaud@gmail.com
:Status:  Draft
:Version: QGIS 2.7+
:Sponsor: SIGE
:Sponsor URL: www.sige.ch


.. note::

    See :ref:`QEP 1 <qep1>` for description of QEP process.

1. Summary
----------

As of today, 9 QGIS plugins_ are tagged with "CAD", proving how the topic is important for users.
We can distinguish plugins that allow creating new objects by offering new map tools or new construction tools from the plugin that allows adding CAD constraints on existing map tools.

This QEP proposes the introduction of advanced digiting tools (aka CAD tools) by porting the CADinput_ plugin in QGIS application.

The general idea is to allow digitizng with precise numerical constraints such as angle, distance and coordinates and top of already existing map tools.
For a better understanding, see the video demo_ of the plugin.


The port has already been done and a pull_ request is associated.

.. _plugins: https://plugins.qgis.org/plugins/tags/cad/
.. _CADinput: https://plugins.qgis.org/plugins/CadInput
.. _pull: https://github.com/qgis/QGIS/pull/1624
.. _demo: https://vimeo.com/85052231

2. Proposed technical Solution
------------------------------

To allow the use of advanced digitizing tools in existing map tools, the idea is to use inheritance from a new map tool class: QgsMapToolAdvancedDigitizing.
Then, subclasses can allow the use of advanced digitzing tools or not.

Using CAD tools requires working on the mouse event but in map coordinates. 
This is required to apply constraints such as distance or coordinates.
To do so, QgsMapToolAdvancedDigitizing redirects all QgsMapTool's virtual events methods (canvasPressEvent, canvasReleaseEvent, canvasMoveEvent,canvasDoubleClickEvent)
to new virtual methods using a new QGIS event class wich integrates the map coordinates of the event, and can also perfom the snapping.
In other words, instead of reimplementing QgsMapTool::canvasReleaseEvent( QMouseEvent* e )
one will reimplement QgsMapToolAdvancedDigitizing::canvasMapReleaseEvent( QgsMapMouseEvent* e )
The new class QgsMapMouseEvent inherits QMouseEvent and provides access to the event in map coordinates and to snapping information.

The new advanced digitizing tools are used from a dock, this new class is named QgsCadDockWidget. 
It has roughly the same GUI as the CADinput plugin.
QgsCadDockWidget is responsible of both the interaction with the user and of the application of the constraints.
This implementation has been chosen since both operations interact with the constraints (angle/distance/x/y/parallel/orthogonal). 
This implementation makes the code much more readable and avoid many signal/slot connection.

To draw construction intermediate points and lines, a new canvas item has been added, QgsCadPaintItem,
directly inheriting from QgsMapCanvasItem.
A new class was addded since no current (QgsRubberBand or QgsHighlight) offered the capacity of drawing circles (for distance constraints). 
It is also quite specific to this use case.
It has a pointer the QgsCadDockWidget to know current applied constraints and so.


3 Specifications
----------------

3.1 Features
............

This integrates all features from the CADinput plugin

* apply constraint on x / y / distance / angle in absolute or relative values.
* additional constraint: orthogonal or parallel to an existing segment
* construction mode: allow placing intermediate points while digitizing (these will not be used as digitized points, they are just helpers for the construction)
* handy UI with keyboard shortcuts (<a>: angle, <d>: distance, <x>,<y>, <enter> locks the constraint, <p> parallel/orthogonal, CTRL/ALT+<a/d/x/y> toggles the lock of the constrait, SHIFT+<a/d/x/y> toggles the relative buttons.
* simple calculations in the line edit (using expressions)

It also provide new features:

* soft snapping to multiple of 90° angles (can be disabled)
* combination of coordinate constraint with distance or angle constraints
* disable/enable snapping


3.2 Restrictions
................

The tools are not enabled if the map canvas is in geographic coordinates.


4. Performance Implications
---------------------------

This is using snapping on mouse move. Thus, it might lead to some poor performance on large projects. 
Although tested without troubles on 30+ layers project with snapping enabled, and some with 30'000+ elements.

However, a snapping refactoring project is about to be started with Lutra Consulting. This should greatly improve the situation.


5. Further Considerations/Improvements
--------------------------------------

At the moment the map tool and the dock are in app, thus not allowing a plugin map tool to enable advanced digitizing tools.
In the future, these could be moved to core/gui to allow subclassing and maybe dealing differently with the constraints.

Once correctly established, CAD specific functions shall be moved to core and be better integrated with the snapping.
This will probably occur in next release cycle when snapping has been refactored.

Open questions: now, tools are enabled in any non geographic map srs. Is this correct? 
Should it be available in geographic coordinates? for projection with important deformation (e.g. pseudo mercator)?

6. Documentation
----------------

A documentation will be issued shortly, especially for keyboard shortcuts. 
The idea is to have a small window to show the shortcuts, and it would be directly accessible from the dock.

7. Issue Tracking ID(s)
-----------------------

(required)


8. Voting History
-----------------

(required)
