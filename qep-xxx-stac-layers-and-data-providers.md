# QGIS Enhancement: STAC layers/data providers

**Date** 2025/02/24

**Author** Jan Dalheimer (@02JanDal)

**Contact** jan.dalheimer at sweco dot se

**Version** QGIS 3.xx

# Summary

QGIS gained built-in support for STAC in 3.40, which was further extended with more functionality in 3.42. One usability improvement we'd like to make is allowing adding STAC endpoints/collections as layers.

This functionality would live roughly at the intersection of the new STAC support, the OGC API Features data provider and the VPC data provider, re-using code from each of them. From a user point-of-view the starting point will be right clicking on either a STAC endpoint or a STAC collection in the Browser, which gives the option to "Add as layer". Clicking this, or dragging the tree item to the map or layer panel, would add a "STAC layer". As a starting point, a STAC layer would function identically to a OGC API Features layer.

Next, there are several more or less orthogonal functionalities to be added:

* **Preview of CO\*-assets** - VPC (Virtual Point Cloud) layers work by being an "index" of streamable point cloud files (such as COPC), but are limited to the point clouds listed in a specific file. The STAC layer would work similarly but show the assets available within it. This means that when sufficiently zoomed in, assets that can be streamed (COG, COPC, EPT) would be shown as a sublayer. In some cases it might additionally be possible to use the overview asset (if available) to render a preview at even higher zoom levels.
* **Download of assets** - It would be natural for users if they could download an asset directly from the STAC layer (for example after having used the identify tool)
* **Usage in algorithms** -  It would be very interesting from a user point-of-view if a STAC layer consisting of, for example, point clouds, could be directly fed into an algorithm such as the point cloud clip
* **Integration with the Temporal Controller** - In STAC, the temporal aspect is often important. As such it would make sense for a layer using STAC as a data source would integrate with the Temporal Controller, by only showing items that are relevant given the current temporal extent.

## Delimitation

* Support for filters could be added in the future, however as search functionality already exists in the Data Source Manager this proposal does not intend to provide an alternative search interface.
* The current scope of work is limited adding STAC layers and previewing streamable assets, potentially with integration with the Temporal Controller in case there is enough time left. Download of assets and usage in algorithms is outside the scope of work.
* Only STAC endpoints that implement at least _STAC API - Core_, _STAC API - Features_ and _STAC API - Search_ will be handled.

## Proposed Solution

### Extent layer - Basics

As the `/search` endpoint defined by `_STAC API - Search_` is very similar to the `/items` endpoints in _OGC API Features_ it can be implemented in the existing `oapif` data provider. The changes would primarily be focused around `QgsOapifProvider::init()`. For layers showing the contents of a single STAC collection no change is required to the existing data provider. The usage of the `/search` endpoint would be triggered by an API those `/conformance` endpoint and `"links"` indicate the existing of this endpoint, if the data source URI has an empty `typename`.

The `QgsOapifProvider` would also be extended by the proper creation of `QgsVectorDataProviderTemporalCapabilities` when reading from a STAC endpoint, as indicated by the conformance classes.

### Extent layer - Assets

In order to expose assets retrieved from the API to other parts of QGIS they need to be stored with the other feature data. Therefore, a new class `QgsFeatureAsset` will be created, modelled after assets in STAC (`href`, `rel`, `title`, `type`, etc.) and `QgsFeature` will be extended by a `QVector<QgsFeatureAsset> assets` property. This data will be populated by `QgsOapifItemsRequest::processReply()`.

### Extent layer - Identify tool

Initial the assets will only be exposed in the identify tool. The `QgsIdentifyResultsDialog` would be extended with another subtree (sibling of `(Derived)`) with one row per asset. That subtree would only be shown for features containing assets. Clicking an item in this subtree will present the user with the choices "Download" and "Add to map" (for assets where this is relevant). In the future this may also be exposed via right click, or drag and drop for adding to the map.

### Point cloud layer

The `vpc` provider already provides rendering of point clouds indexed by a STAC-aligned format. It will be extended by the ability to optionally load the list of available point clouds dynamically based on the current map extent. As far as possible the network functionality should re-use parts of the `oapif` provider, such as the `QgsOapifItemsRequest`.

When fed into a processing algorithm a first step would require downloading all items regardless of extent. This may be suboptimal, as for example the clip algorithm could in theory provide its down bounding box, however this optimization is kept out-of-scope for the current proposal.

### Raster layer

TBD, out of scope for this proposal. Might use the GDAL `STACIT` driver.

### Integration with STAC browser

Additional menu items needs to be added for endpoints and collections in `QgsStacDataItemGuiProvider`, which adds a layer to the map using the appropriate data source URIs, either just based on the extent of the items or rendering raster or point cloud data.

### Layers with fixed configuration

As STAC pre-defines some properties that are usually user-changable there needs to be a mechanism to prevent the user from making changes. For example, for temporal filtering support, we'd as part of creating the STAC layer (such as a `QgsVectorLayer`) create a `QgsVectorLayerTemporalProperties` based on the settings from the data provider. However, this does not prevent a user from changing this configuration, which probably shouldn't be possible.

For now, this will be allowed, however a future iteration may introduce a way for the data provider to restrict changing certain layer configuration.

<!--## Deliverables-->

<!--### Example(s)-->

<!-- ### Affected Files -->

## Risks

None known.

## Performance Implications

Similar issues as with general OGC API Feature layers; as there is no generalization involved viewing such a layer at a very low zoom level could result in large amounts of data being fetched. However, STAC endpoints in general contain larger features, so the issue should be less prononced than with OGC API Feature layers.

<!--## Further Considerations/Improvements

*(optional)*

## Backwards Compatibility

**(required if applicable)**

## Issue Tracking ID(s)

*(optional)*-->
