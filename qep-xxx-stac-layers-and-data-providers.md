# QGIS Enhancement: STAC layers/data providers

**Date** 2025/02/24

**Author** Jan Dalheimer (@02JanDal)

**Contact** jan.dalheimer at sweco dot se

**Version** QGIS 3.xx

# Summary

QGIS gained built-in support for STAC in 3.40, which was further extended with more functionality in 3.42. One usability improvement we'd like to make is allowing adding STAC endpoints/collections as layers.

This functionality would live roughly at the intersection of the new STAC support, the OGC API Features data provider and the VPC data provider, re-using code from each of them. From a user point-of-view the starting point will be right clicking on
either a STAC endpoint or a STAC collection in the Browser, which gives the option to "Add as layer". Clicking this, or dragging the tree item to the map or layer panel, would add a "STAC layer". As a starting point, a STAC layer would function
identically to a OGC API Features layer, except with more limited styling functionality (possibly completely static).

Next, there are several more or less orthogonal functionalities to be added:

* **Preview of CO\*-assets** - VPC (Virtual Point Cloud) layers work by being an "index" of streamable point cloud files (such as COPC), but are limited to the point clouds listed in a specific file. The STAC layer would work similarly but show the assets available within it. This means that when sufficiently zoomed in, assets that can be streamed (COG, COPC, EPT) would be shown as a sublayer. In some cases it might additionally be possible to use the overview asset (if available) to render a preview at even higher zoom levels.
* **Download of assets** - It would be natural for users if they could download an asset directly from the STAC layer (for example after having used the identify tool)
* **Usage in algorithms** -  It would be very interesting from a user point-of-view if a STAC layer consisting of, for example, point clouds, could be directly fed into an algorithm such as the point cloud clip
* **Integration with the Temporal Controller** - In STAC, the temporal aspect is often important. As such it would make sense for a layer using STAC as a data source would integrate with the Temporal Controller, by only showing items that are relevant given the current temporal extent.

## Delimitation

* Support for filters could be added in the future, however as search functionality already exists in the Data Source Manager this proposal does not intend to provide an alternative search interface.
* The current scope of work is limited adding STAC layers and previewing streamable assets, potentially with integration with the Temporal Controller in case there is enough time left. Download of assets and usage in algorithms is outside the scope of work.
* Initially, only STAC endpoints that implement at least _STAC API - Core_, _STAC API - Features_ and _STAC API - Search_ will be handled. This may be relaxed in the future.
* By using the same STAC handling (`QgsStac*`) as the existing STAC support the same limitation of support STAC versions will apply (and conversely, support for additional versions would benefit both parts of the code base).

## Proposed Solution

The core of this functionality would be a new data provider with key `stac`, those data source URI takes the same parameters as `QgsStacConnection` with the addition of an optional `collection` parameter.

Additional menu items needs to be added for endpoints and collections in `QgsStacDataItemGuiProvider`, which adds a layer to the map using the data source URI for the aforementioned data provider, either just based on the extent of the items or rendering raster or point cloud data.

### STAC data provider

There was a bit of a design decision if this functionality should be built based on the OGC API Features provider (which makes re-use of the existing `QgsStac*` classes harder, and might instead require re-implementing some of the STAC-specific parsing) or be built from scratch
using the `QgsStac*` classes (which would focus all STAC-specific functionality, but require some more work in setting up the data provider). The current plan is to go with the later option, as in practice the STAC data provider is likely to have more in common with the `QgsStac*` classes than the
OGC API Features provider (which also would bring in a lot of functionality not needed for STAC, such as editing). This also enables supporting STAC endpoints that do not conform to OGC API Features (which, from the point-of-view of STAC, is optional).

As such, a new `QgsStacDataProvider`, subclassing `QgsVectorDataProvider`, would be added together with the required "companion" classes (`QgsStacFeatureIterator`, `QgsStacFeatureSource`, etc.). These classes use the `QgsStac*` classes to access the STAC API, such as `QgsStacController`, `QgsStacItemCollection` and `QgsStacItem`.
A new class will be added that handles construction of STAC API search requests (which can then be passed to `QgsStacController::fetchItemCollection`/`QgsStacController::fetchItemCollectionAsync`), tentatively named `QgsStacSearchQueryBuilder`.

### STAC raster and point cloud data providers

Additionally, in order to support previewing of cloud optimized assets, we'll need `QgsRasterDataProvider` and `QgsPointCloudDataProvider` subclasses (`QgsStacRasterDataProvider` and `QgsStacPointCloudDataProvider`). They'll be implemented similarly to the extent-only `QgsStacDataProvider`, but using the sublayer-functionality to
render rasters/point clouds at appropriate zoom-levels. `QgsVirtualPointCloudProvider` should be usable as a source to much of this (sublayer-related) functionality.

It would have been beneficial if we could have the same subclass for both extent, raster and point cloud data providers, however we'd risk running into the Diamond Inheritance problem so that is not an option.

For these data providers we'll make use of some [STAC extensions](http://stac-extensions.github.io/) where available, notably Classification, Point Cloud, Projection and Raster.

### STAC layers

Initially, we'll just use `QgsVectorLayer`, `QgsRasterLayer` and `QgsPointCloudLayer`.

There are some upsides as well as downsides with this approach as we'd be using most of the existing plumbing from these layer classes. For example this means that STAC layers can be used in processing algorithms, which may or may not be advisable, but would not be impossible. A bigger drawback is that it is not currently
possible (AFAICT) to "lock down" configuration of layers. For example, for temporal filtering support, we'd as part of creating the STAC layer (such as a `QgsVectorLayer`) create a `QgsVectorLayerTemporalProperties` based on the settings from the data provider. However, this does not prevent a user from changing this configuration, which
probably shouldn't be possible.

However, while it might longer term thus be a good idea to have either separate layer classes for STAC or alternatively introduce a way for data sources to limit what is user-configurable (both of which would be significant undertakings), I'd prefer to keep it outside the scope of the current proposal.

<!--## Deliverables-->

<!--### Example(s)-->

<!-- ### Affected Files -->

## Risks

None known.

## Performance Implications

Similar issues as with OGC API Feature layers; as there is no generalization involved viewing such a layer at a very low zoom level could result in large amounts of data being fetched. However, STAC endpoints in general contain larger features, so the issue should be less prononced than with OGC API Feature layers.

<!--## Further Considerations/Improvements

*(optional)*

## Backwards Compatibility

**(required if applicable)**

## Issue Tracking ID(s)

*(optional)*-->
