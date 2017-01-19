.. _qep#[.#]:

========================================================================
QGIS Enhancement 9: Map Rotation Support
========================================================================

:Date: 2014/12/13
:Author: Sandro Santilli
:Contact: strk@keybit.net
:Last Edited: 2014/12/13
:Status:  Draft
:Version: QGIS 2.7+
:Sponsor: (optional) Sponsor Name - State/Province/Region, Country

.. note::

    See :ref:`QEP 1 <qep1>` for description of QEP process.

#. Summary
----------

Some applications (for example navigation) require that the map view
is rotated to follow direction of movement. Such rotation needs to be
performed dynamically and cannot be a static parameter of the output
map; it would be very similar to the "scale" and "position" parameters
of a map view.

This enhancement proposal is for adding rotation support to the QGIS
map canvas object and ensuring standard navigation and editing tools
still work correctly in presence of rotation.

#. Proposed Change
------------------

Map configuration should be changed from being defined by "extent"
to being defined by a triplet of "center", "resolution" and "rotation".

Code should stop assuming that an extent is enough to fully define
a map view but rather rely on the whole triplet.

#. Implementation Details
-------------------------

#.# QgsMapToPixel class changes
...............................

The QgsMapToPixel class, used to convert from map and device coordinate
spaces, needs to be changed to allow for specifying a rotation and a
center around which to rotate the map. At the device level we can enforce
rotation to always happen at the middle of the output so all we need to
know is the rotation value, the center of the map (in map units)
and the size of the output (in pixels).

The class will get an new constructor and an in-place setter taking all
required parameters:

  * mapUnitsPerPixel Map units per pixel
  * xc X ordinate of map center, in geographical units
  * yc Y ordinate of map center, in geographical units
  * width Output width, in pixels
  * height Output height, in pixels
  * degrees clockwise rotation in degrees

#.# QgsMapSettings class changes
................................

The QgsMapSettings class will get new methods to set/get center
and rotation:

  * setRotation / rotation -- clockwise rotation in degrees
  * setCenter / center     -- in map units

#.# QgsMapCanvas class changes
..............................

The QgsMapCanvas class will get new methods to set/get center
and rotation:

  * setRotation / rotation -- clockwise rotation in degrees
  * setCenter / center     -- in map units

#. Backwards Compatibility
--------------------------

Existing map configuration function will be retained to mean
no rotation is in place.

#. Issue Tracking ID(s)
-----------------------

(required)

#. References
-------------

(optional)

#. Miscellaneous
----------------

(optional)

#. Voting History
-----------------

(required)
