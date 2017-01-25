.. _qep#[.#]:

========================================================================
QGIS Enhancement #: Layout rework (composer v2)
========================================================================

:Date: 2014/11/16
:Author: Nyall Dawson
:Contact: nyall dot dawson at gmail dot com
:Last Edited: 2014/11/16
:Status:  Draft
:Version: QGIS 3.0
:Sponsor: TBC

.. note::

    See :ref:`QEP 1 <qep1>` for description of QEP process.

1. Summary
----------

QGIS' current composer functionality has a number of significant design limitations.
It is proposed that the composer functionality be rebuilt to overcome these limitations,
and to create a better foundation for this feature moving forward.

2. Summary of current issues/limitations
----------------------------------------

The composer functionality suffers from a substational number of design and API limitations.
Some of these include:

- Composer logic is tied up across the entire codebase, with logic being spread across core, gui and app.
  This makes features fragile, makes bug fixing difficult, and makes it very difficult for plugins to manipulate
  and export compositions without reimplementing large blocks of QGIS code within the plugin.

- There is large amounts of item-specific logic and handling scattered through QgsComposition, QgsComposerView and
  QgsComposer. This makes it impossible to have features like plugin generated item types, and makes maintenance difficult.

- Everything is coded to expect measurements and sizes in mm. It's not possible to implement other units (such as pixels
  or inches) without breaking api and resorting to a lot of hacks. (see #7367, #7959)

- Compositions which consist of mixed page sizes and orientation require an API and significant refactoring
  to implement.

- Undo/redo support is extremely fragile, and leads to hard crashes. This is impossible to fix without 
  a large rework of how items are handled within compositions. (see #11371)

- QgsComposition should not require a QgsMapSettings/QgsMapRenderer. This should instead be set individually for map items.
  Doing so would pave the way for features such as reprojection support for individual map items.

- Items currently utilise the paint method for non-rendering purposes, such as setting a minimum or fixed size for the item. This
  prevents QGIS from utilising the existing caching methods for QGraphicsItems, leading to slower operation of the composer.

- The composition code is full of deprecated methods and legacy api. Over time the requirements of compositions
  have changed significantly, and the code is now quite messy and difficult to maintain. Moving forward, this will only get worse
  if the code is not refactored substationally.


3. Proposed Solution
--------------------

The composer will be rebuilt to address these design limitations.

4. Implementation Details
-------------------------

New classes will be created:

- QgsLayoutUnit: base class for layout units, including page units such as millimetres and inches, and screen units (pixels)

- QgsLayoutMeasurement, QgsLayoutSize, QgsLayoutPoint: classes for single dimension measurements, sizes and points respectively.
  These classes will store the unit type alongside the measurement.

- QgsLayoutMeasurementConverter: handles conversion between different layout units. For conversion between screen and print
  units, QgsLayoutMeasurementConverter will utilise a dpi (dots per inch) property.

- QgsLayout: this will form the base class for all layouts, including both print compositions and report sections. Notable differences
  from QgsComposition include:

    - QgsLayout does not require a QgsMapSettings object.
    
    - Items removed from a layout are immediately deleted. Undo/redo functionality will work by storing/restoring an item's xml content,
      rather than a pointer to the item itself.
    
    - All export and item handling logic will be moved from QgsComposerView and QgsComposer to QgsLayout, with the exception of
      atlas export logic (which will be moved to QgsLayoutAtlas)
    
- QgsPrintLayout (or "QgsCompositionV2" - see note below): inherits from QgsLayout, with notable differences from QgsComposition including:

    - QgsPrintLayout will maintain a QgsPageCollection, which can consist of multiple pages with different size and orientation

- QgsLayoutContext: will store rendering flags (such as whether antialiasing should be enabled for the layout (#9281) ). QgsLayoutContext
  will also store pointers to the current feature and associated vector layer for a layout. For print layouts setting the current
  feature and layer will be handled by the atlas.

- QgsLayoutObject: base class for layout objects which can have data defined properties. This includes both layout items,
  and components of items (for instance, an individual map grid will be a QgsLayoutObject, to allow grid properties to be data defined).

- QgsLayoutItem: base class for layout items. Notable differences from QgsComposerItem include:

   - items are not expected to override the paint method, but instead to implement a pure virtual method "draw". All non-drawing 
     code (such as item resizing) will be seperated from the painting code. New methods will be added for setting a minimum or 
     fixed size for the item. New methods will be added "attemptResize" and "attemptMove", which will form a single code path for
     setting an item's size or position which respects overrides such as the item's reference point, minimum size, data defined overrides,
     etc.
   
   - item's will enable Qt's graphics scene caching via a "setCacheMode( QGraphicsItem::DeviceCoordinateCache )" call
   
   - methods and properties specific to rectangular items will be removed

- QgsLayoutRectItem: inherits from QgsLayoutItem, and adds methods specific to rectangular items, such as frame and background settings.

- QgsLayoutItemRegistry: singleton which handles registration of new items types (eg, plugin provided item types), also functions
  as a factory for creating new items

- QgsPageSizeRegistry: singleton which maintains a registry of known page sizes


5. Reporting framework
----------------------

This proposed rebuild is also being driven by the planned work on adding a reporting framework into QGIS. This will be the subject
of an additional QEP.

6. Naming
---------

Depending on the outcome of QEP #7 - Rename Composer, the reworked print layout class will either be QgsPrintLayout or QgsCompositionV2.

7. Python Bindings
------------------

Will be updated as required.

8. Affected Files
-----------------

This work will be implemented in parallel to the current composer code. All work will consist of new classes, and the existing composer
code and API will remain until PSC agree to switch over to the new layout code.

9. Test Coverage
----------------

Since this will be implemented as a ground-up rebuild, we will aim for 100% unit test coverage for non GUI code.

10. Backwards Compatibility
---------------------------

Layouts will not be backwards compatible with older QGIS versions. Compositions will be automatically upgraded to layouts
on project load.

11. Voting History
------------------

(required)
