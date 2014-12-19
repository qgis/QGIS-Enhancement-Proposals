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

A new QgsLayerTreeModelLegendNode subclass specific to WMS legend
will be added (QgsWMSLegendNode).

The default raster layer legend tree model constructor will create an instance
of the new QgsWMSLegendNode class when asked to create the legend nodes for
raster layers using the WMS provider
(QgsDefaultRasterLayerLegend::createLayerTreeModelLegendNodes).

The QgsWMSLegendNode instances will lazily request the WMS legend when first
asked for and cache it.  The invalidateMapBasedData() virtual method of
QgsWMSLegendNode will clear the cache.

The QgsLayerTreeModel will invoke invalidateMapBasedData() for each legend
node left after filtering due to QgsLayerTreeModel::setLegendFilterByMap.
(this part is not strictly related to the new feature, but would be a
possible optimization).

Layer configuration for enabling/disabling contextual WMS legend
will be implemented at the provider level, similarly to configuration
for tile size and image format.

A virtual method to asynchronously fetch legend graphic will be added to
the QgsRasterDataProvider class. The method will accept an optional pointer
to a QgsMapSetting and return a newly created QgsImageFetcher object that
will signal events on complete download, error and progress.
The legend node will handle those events to update the legend item accordingly.

#.# Affected Files
..................

 src/app/qgisapp.cpp                                |   4 +-
 src/core/layertree/qgslayertreemodel.cpp           |  10 ++-
 src/core/layertree/qgslayertreemodel.h             |   2 +-
 src/core/layertree/qgslayertreemodellegendnode.cpp | 121 ++++++++++++++++++++++++++++
 src/core/layertree/qgslayertreemodellegendnode.h   |  45 +++++++++++
 src/core/qgsmaplayerlegend.cpp                     |   7 +-
 src/core/raster/qgsrasterdataprovider.h            |  58 +++++++++++++-
 src/providers/wms/qgswmscapabilities.cpp           |   3 +
 src/providers/wms/qgswmscapabilities.h             |   2 +
 src/providers/wms/qgswmsprovider.cpp               | 382 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++------------------------
 src/providers/wms/qgswmsprovider.h                 | 103 ++++++++++++++++++++++--
 src/providers/wms/qgswmssourceselect.cpp           |   2 +
 src/ui/qgswmssourceselectbase.ui                   |  12 ++-

#. Performance Implications
---------------------------

Having WMS layers with contextual legend enabled and visible
would trigger a new legend download on every pan/zoom operation.

#. Backwards Compatibility
--------------------------

Backward compatibility will be retained.

#. Issue Tracking ID(s)
-----------------------

http://hub.qgis.org/issues/11859

#. References
-------------

Mapserver's RFC-101: http://www.mapserver.org/development/rfc/ms-rfc-101.html

#. Voting History
-----------------

(required)
