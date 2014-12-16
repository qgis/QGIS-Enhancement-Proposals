.. _qep#[.#]:

========================================================================
QGIS Enhancement 10: Support for contextual WMS legend
========================================================================

:Date: 2014/12/13
:Author: Sandro Santilli
:Contact: strk@keybit.net
:Last Edited: 2014/12/13
:Status:  Draft
:Version: QGIS 2.7+
:Sponsor: Regione Toscana, settore sistema informativo territoriale ed ambientale, Italy

.. note::

    See :ref:`QEP 1 <qep1>` for description of QEP process.

#. Summary
----------

Some WMS servers support an extension to the GetLegendGraphics request
to make the legend dependent on the extent and scale of the view.

This proposal is to add support for such servers in QGIS.

The specification of the extended server API exists as an RFC of
Mapserver: http://www.mapserver.org/development/rfc/ms-rfc-101.html

#. Proposed Behavior
--------------------

Users shall be allowed to specify if they want to enable contextual
legend in the WMS Layer configuration.

Legend items for WMS layers where contextual legend is enabled should
be updated whenever the map setting changes, but only if their legend
is visible and map-based filtering is also active for the whole
layertree.

#. Implementation Details
-------------------------


#.# Example(s)
..............

(optional)

#.# Python Bindings
...................

(required if applicable)

#.# Affected Files
..................

(required if applicable)

#. Test Coverage
----------------

(required for technical solutions/changes if applicable)

#. Performance Implications
---------------------------

(required if applicable)

#. Further Considerations/Improvements
--------------------------------------

(optional)

#. Restrictions
---------------

(optional)

#. Backwards Compatibility
--------------------------

(required)

#. Documentation
----------------

(required if applicable)

#. Issue Tracking ID(s)
-----------------------

http://hub.qgis.org/issues/11859

#. References
-------------

Mapserver's RFC-101: http://www.mapserver.org/development/rfc/ms-rfc-101.html

#. Voting History
-----------------

(required)
