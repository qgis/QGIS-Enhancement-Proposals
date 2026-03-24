# QGIS Enhancement:  Layer Name in GetFeatureInfo JSON Result as `featureType`

**Date** 2026/02/03

**Author** Dave Signer (@signedav)

**Contact** david at opengis dot ch

**Version** QGIS 4.2

## Summary

When performing a `GetFeatureInfo` request, QGIS Server returns features for the requested position, layer, etc.

While e.g. the XML format provides clear information about what layer the feature belongs to by passing the name (short-name), the **GeoJSON response lacks a reliable information to identify the layer name.**

We are looking for a way to provide the layer information within the response without violating the [GeoJSON](https://datatracker.ietf.org/doc/html/rfc7946) standard and propose **enabling the passing of the layer name as a member called `featureType` as [defined in the OGC specification for JSON FG](https://portal.ogc.org/files/107269#feature-types).**

## Current Behavior / Problem

The response for `INFO_FORMAT=text/xml` explicitly wraps features in a `layer` element, providing the name (short-name):

```xml
<GetFeatureInfoResponse>
<Layer name="whata.layer" title="What a Layer">
<Feature id="1">
<Attribute name="theid" value="1"/>
<Attribute name="name" value="test"/>
<Attribute name="oid" value="2"/>
<Attribute name="somenumber" value="layer.1"/>
</Feature>
</Layer>
</GetFeatureInfoResponse>
```

The response for `INFO_FORMAT=application/json` returns a flat `FeatureCollection`. The only hint regarding the source layer is usually concatenated within the `id` field:

```json
{
  "features": [
    {
      "geometry": null,
      "id": "whata.layer.1",
      "properties": {
        "name": "test",
        "oid": 2,
        "somenumber": "layer.1",
        "theid": 1
      },
      "type": "Feature"
    }
  ],
  "type": "FeatureCollection"
}
```

QGIS Server generates the `id` by joining the layer name and the feature ID (usually the primary key) with a dot. In our case, this includes "whata.layer" (layer name) and "1" (our primary key column `theid`).

While this provides some kind of information, a client cannot rely on it because both the layer name and the feature ID (primary key) can contain dots. This makes it impossible to parse the string accurately to determine where the layer name ends and the feature ID begins. And, since a client cannot know which property represents the primary key, it cannot simply cut it from the end of the `id` object.

## Proposed Solution

QGIS should return the layer name (short name) in the `featureType` member introduced in the ["OGC Features and Geometries JSON" (JSON-FG) specification](https://portal.ogc.org/files/107269). In GeoJSON, this additional member is permitted as a [foreign member](https://datatracker.ietf.org/doc/html/rfc7946#section-6.1).

A `featureType` can be returned for the entire `FeatureCollection` ([when it is the same for all features](https://portal.ogc.org/files/107269#types-schemas_feature-type)) or per `feature`. Since QGIS returns a "flat" `FeatureCollection` for all the requested features of the requested layers, we suggest returning it per `feature`.

The result will then look like this:
```json
{
  "features": [
    {
      "featureType": "whata.layer",
      "geometry": null,
      "id": "whata.layer.1",
      "properties": {
        "name": "test",
        "oid": 2,
        "somenumber": "layer.1",
        "theid": 1,
        "layer_name": "whata.layer"
      },
      "type": "Feature"
    }
  ],
  "type": "FeatureCollection"
}
```

We suggest returning the `featureType` member containing the layer name in any case. We do not see value in introducing a new setting or WMS parameter.

## Implementation

In [`QgsRenderer::convertFeatureInfoToJson`](https://github.com/qgis/QGIS/blob/master/src/server/services/wms/qgswmsrenderer.cpp#L3115) the layer name should be passed to [`QgsJsonExporter::exportFeatureToJsonObject`](https://github.com/qgis/QGIS/blob/master/src/core/qgsjsonutils.h#L241):

```c++
  json["features"].push_back( exporter.exportFeatureToJsonObject( feature, extraProperties, id, layerName ) );
```

Where it can be appended to the member `featureType`.

### Deliverables

- Functionality in `QgsRenderer::convertFeatureInfoToJson`
- Functionality in `QgsJsonExporter::exportFeatureToJsonObject`

### Affected Files

- src/server/services/wms/qgswmsrenderer.cpp
- src/core/qgsjsonutils.h
- src/core/qgsjsonutils.cpp

## Risks

None expected.

## Performance Implications

None expected.

## Backwards Compatibility

Ensured.