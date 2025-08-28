# QGIS Enhancement: Opaque Layergroups in QGIS-Server

**Date** 2025/08/28

**Author** Oliver Jeker (@ojeker)

**Contact** oliver dot jeker at bd dot so dot ch

**Version** QGIS 4.X.X

# Summary

We would like to extend QGIS with the capability to serve opaque layergroups through OGC WMS. Opaque layergroups hide all their children, grandchildren, ... in the layertree.

Our motivation is to hide implementation details from the user. One usage example for an opaque layergroup is a detailed road_polygon dataset and an overview road_line dataset. The point and polygon layer of these two datasets are contained in the opaque layergroup "roads".

## Intended behaviour

Most behaviour of an opaque layergroup is the same as in the "normal" already existing layergroup. This chapter explains the differences in behaviour between opaque and normal layergroups for all WMS operations.

Differing behaviour for:

* GetCapabilities:   
Opaque layergroups hide all their children, grandchildren, ... in the GetCapabilities response. Only the opaque layergroup itself is returned and looks like a "singlesource" layer (QgsVectorLayer, QgsRasterLayer) in the layer tree.
* GetMap request on children:  
Requests containing a child, grandchild, ... in the "layers" parameter get the same error response as when requesting a layer that does not exist in the layer tree.   

No difference in behaviour between normal and opaque layergroups for:

* GetMap request on the opaque layergroup:   
Requests containing opaque layergroups in the "layers" parameter are rendered the same way as normal layergroups. 
* GetFeatureInfo
* DescribeLayer
* GetLegendGraphic
* GetStyle(s)

### Notes on duplicate layer names in the layer tree

For a configuration containing a layer with the same name both as child of an opaque layer and in a non-opaque part of the layer tree:

When requesting the layer name, only the layer in the non-opaque part of the layer tree must be rendered.

When requesting the opaque layer, only the child of the opaque layer must be rendered.

### Note on access management

We plan to configure the access management on the level of the singlesource layer's and aggregate it up to the corresponding group layer's. 

A group layer containing several public layer's and one secret layer will only be served to the users with permission on the secret layer.

This chapter is for information only - we see no need to implement anything concerning access management for this QEP.

## Proposed Solution

Add a checkbox or dropdown to the form "Set Group WMS Data" to choose if the group is opaque or not. Rename the form to "Set Group WMS Properties" or similar.

![group_wms_data.png](images/qep402/group_wms_data.png)