# QGIS Enhancement:  Layer Name in GetFeatureInfo JSON Result

**Date** 2026/02/03

**Author** Dave Signer (@signedav)

**Contact** david at opengis dot ch

**Version** QGIS 4.2

# Summary

When performing a `GetFeatureInfo` request, QGIS Server returns features for the requested position, layer, etc.

While e.g. the XML format provides clear information about what layer the feature belongs to by passing the name (short-name), the **GeoJSON response lacks a reliable information to identify the layer name.**

We are looking for a way to provide the layer information within the response without violating the [GeoJSON](https://datatracker.ietf.org/doc/html/rfc7946) standard and propose **enabling the passing of the layer name as a property.**

# Current Behavior / Problem

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

# Proposed Solution

A [new QGIS Server extra parameter](https://docs.qgis.org/3.44/en/docs/server_manual/services/wms.html#getfeatureinfo) should be introduced: `WITH_LAYER_NAME_PROPERTY=layer_name`.

```c++
const QgsWmsParameter pWithLayerNameProperty( QgsWmsParameter::WITH_LAYER_NAME_PROPERTY, QMetaType::Type::QString );
```

The two get-methods check if the parameter was passed and, if so, what the desired property should be called:
```c++
QString withLayerNamePropertyAsString() const;
bool withLayerNameProperty() const;
```

In [`QgsRenderer::convertFeatureInfoToJson`](https://github.com/qgis/QGIS/blob/master/src/server/services/wms/qgswmsrenderer.cpp#L2985), the option should be checked and, if true, another `extraProperty` should be added.

Similar to the existing [`withDisplayName` property](https://github.com/qgis/QGIS/blob/master/src/server/services/wms/qgswmsrenderer.cpp#L3111-L3114) we would introduce the new functionality:

```c++
if ( mWmsParameters.withLayerNameProperty() )
{
  extraProperties.insert( mWmsParameters.withLayerNamePropertyAsString(), layerNickname(vl) );
}
```

The result will then look like this:
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
        "theid": 1,
        "layer_name": "whata.layer"
      },
      "type": "Feature"
    }
  ],
  "type": "FeatureCollection"
}
```

To ensure consistency, the new extra attribute should be included in the other formats as well as the JSON format. This part of the implementation will be located in [`QgsRenderer::featureInfoFromVectorLayer`](https://github.com/qgis/QGIS/blob/master/src/server/services/wms/qgswmsrenderer.cpp#L1845) and results in the following XML:

```xml
<GetFeatureInfoResponse>
<Layer name="whata.layer" title="What a Layer">
<Feature id="1">
<Attribute name="theid" value="1"/>
<Attribute name="name" value="test"/>
<Attribute name="oid" value="2"/>
<Attribute name="somenumber" value="layer.1"/>
<Attribute name="layer_name" value="whata.layer"/>
</Feature>
</Layer>
</GetFeatureInfoResponse>
```

### Name of the extra attribute

The name of the extra property can be passed via the parameter. However, if the client passes `true`, `on`, `yes` or `1`, the default should align with the current `display_name`/`DisplayName` like this: `layer_name`/`LayerName`.

### Name collision with feature attribute name

This concerns only JSON, not XML etc. where multiple same-named values are allowed. To keep valid GeoJSON, we cannot return same-named properties. Therefore, in the event of a collision, the feature attribute is ignored and the extra property is prioritized. This aligns with the behavior of `WITH_DISPLAY_NAME` (there is still a [bug](https://github.com/qgis/QGIS/issues/65004) but it does not concern this QEP).

### Why new WMS parameter instead of a servers side setting.

The question is whether a QGIS-specific behavior should be triggered by [a QGIS Server extra parameter](https://docs.qgis.org/3.44/en/docs/server_manual/services/wms.html#getfeatureinfo) or a server-side setting. **We prefer the WMS parameter**. This is because some clients could require this information while others do not, yet they still request from the same server. Another reason is that, with this approach, existing clients that do not need this info will never encounter unasked information.

## Deliverables

- New WMS parameter and methods in `QgsWmsParameter`
- Functionality in `QgsRenderer::featureInfoFromVectorLayer`
- Functionality in `QgsRenderer::convertFeatureInfoToJson`

### Affected Files

- src/server/services/wms/qgswmsrenderer.h
- src/server/services/wms/qgswmsrenderer.cpp
- src/server/services/wms/qgswmsparameters.h
- src/server/services/wms/qgswmsparameters.cpp

## Risks

The more QGIS-specific WMS parameters, the more complexity.

## Performance Implications

None expected.

## Backwards Compatibility

Completely ensured.