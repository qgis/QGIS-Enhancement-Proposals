# QGIS Enhancement: Opaque Layergroups in QGIS-Server

**Date** 2025/08/28

**Author** Oliver Jeker (@ojeker) / Dave Signer (@signedav)

**Contact** oliver dot jeker at bd dot so dot ch / david at opengis dot ch

**Version** QGIS 4.X.X

# Summary

We would like to extend QGIS with the capability to serve opaque layergroups through OGC WMS. Opaque layergroups hide all their children, grandchildren, ... (named opaque childlayers in this QEP) in the layertree. In [GeoServer](https://docs.geoserver.org/main/en/user/data/webadmin/layergroups.html#layer-group-modes) it's called an opaque container, while in QGIS we would use "opaque" as the descriptive word as well and keep layer groups to follow QGIS-style wording.

Our motivation is to hide implementation details from the user. One usage example for an opaque layergroup is a detailed road_polygon dataset and an overview road_line dataset. The point and polygon layer of these two datasets are contained in the opaque layergroup "roads".

## Intended behaviour

Most behaviour of an opaque layergroup is the same as in the "normal" already existing layergroup. This chapter explains the differences in behaviour between opaque and normal layergroups for all WMS operations.

### Differing behaviour for

* **GetCapabilities**   
Opaque layergroups hide all their opaque childlayers in the GetCapabilities response. Only the opaque layergroup itself is returned and looks like a "singlesource" layer (QgsVectorLayer, QgsRasterLayer) in the layer tree.

* **GetContext**    
Opaque childlayers are not listed.

* **GetProjectSettings**    
Expose the new property in the group setting information.
    ```xml
    <Layer expanded="1" mutuallyExclusive="0" queryable="1" visibilityChecked="1" visible="1">
        <Name>Roads</Name>
        [...]
        <IsOpaque>1</IsOpaque>
        [...]
    ```

* **Requests on children (like GetMap, GetFeatureInfo, GetStyle(s), GetLegendGraphic(s), DescribeLayer)**    
Requests containing a opaque childlayers in the "layers" parameter get the same error response as when requesting a layer that does not exist in the layer tree.
    ```xml
    <ServiceExceptionReport xmlns="http://www.opengis.net/ogc" version="1.3.0">
        <ServiceException code="LayerNotDefined">The layer 'RoadsInOpaqueGroup' does not exist.</ServiceException>
    </ServiceExceptionReport>
    ```
    What is `OGC_LayerNotDefined` in source code.

#### Is "Layer does not exist" always the preferred feedback?

There is a different behavior on requesting a non-existent layer or an excluded layer.

  Request           | Non-Existent         | Excluded            |
 |---------------------|----------------------|---------------------|
 | GetMap              | OGC_LayerNotDefined  | Not displayed (white canvas)       |
 | GetFeatureInfo      | OGC_LayerNotDefined  | OGC_LayerNotDefined |
 | GetStyle / GetStyles| OGC_LayerNotDefined  | Empty reply      |
 | GetLegendGraphic    | OGC_LayerNotDefined  | OGC_LayerNotDefined | 

We propose to handele the opaque childlayers same like the non-existent layer and return `OGC_LayerNotDefined`. Why is the behavior different on excluded layers?

### Same behaviour for

Normal layergroups and opaque layergroups on:

* **GetMap** request on the opaque layergroup   
Requests containing opaque layergroups in the "layers" parameter are rendered the same way as normal layergroups. 
* **GetFeatureInfo**
* **GetLegendGraphic**
* etc.

### Preferred behavior on duplicate layer names in the layer tree

***Note that it's bad practice to use same named layers in the same QGIS project. If this preferred behavior brings us too much complexity into the code, we would not make it part or the implementation.***

For a configuration containing a layer with the same name both as child of an opaque layer and in a non-opaque part of the layer tree:
* When requesting the layer name, only the layer in the non-opaque part of the layer tree must be rendered.
* When requesting the opaque layer, only the child of the opaque layer must be rendered.

Be aware of the fact, that same named layers usually [get merged](https://github.com/qgis/QGIS/pull/33952).

## Proposed Solution

Add a checkbox or dropdown to the form "Set Group WMS Data" to choose if the group is opaque or not. Maybe we could rename the form to "Set Group WMS Properties" or similar.

![group_wms_data.png](images/qep402/group_wms_data.png)

This would mean there would be a new member `isOpaque` in the `QgsMapLayerServerProperties` of the `QgsLayerTreeGroup`.

This information is accessible via `QgsMapLayerServerProperties *QgsLayerTreeGroup::serverProperties()`.

In the server classes it might be handled similar to the `wmsRestrictedLayers` (the excluded layers), but with the difference, that they are not hidden on everything (e.g. rendered on GetMap requests of the group).

On `QgsWmsRenderContext::initOpaqueGroupLayers()` it checks the `isOpaque` on the groups' server properties and fills up the `QStringList mOpaqueChildLayers`. They should not be considered in `removeUnwantedLayers`, because it still should be rendered. 

On a request on one of the opaque childlayers it should check `QgsWmsParameter::LAYER` against `mOpaqueChildLayers` and throw the `OGC_LayerNotDefined` error "The layer '%1' does not exist."

On `GetCapabilties` it should return in functions like `appendLayersFromTreeGroup` and be detected for removing in `appendDrawingOrder` etc.

### Duplicate layer names handling

The layer name should just not be added to the `mOpaqueGroupLayers`, when there is a same named layer outside an opaque layergroup. On requesting the layer outside the opaque layergroup it should care in `QgsWmsRenderContext::searchLayersToRender()` that it should not render the same named layer inside an opaque layergroup.



