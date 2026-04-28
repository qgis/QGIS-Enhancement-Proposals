# QGIS Enhancement: QGIS Server OAPIF JSON-FG and FlatGeobuf formats

**Date** 2026/03/30

**Author** Alessandro Pasotti

**Contact** elpaso at itopen dot it

**Version** QGIS 4.2

# Summary

This QEP is about adding two new output formats to QGIS Server OAPIF:

- JSON-FG ( see: https://docs.ogc.org/DRAFTS/21-045.html )
- FlatGeoBuf ( see: https://flatgeobuf.org/ )

## Motivation

The main motivation is to provide more output format capabilities to QGIS
Server in order to make it more useful to modern API consumers and possibly to
improve the performances for large datasets.

Other OAPIF implementations already support these formats (e.g. MapServer, GeoServer, ldproxy),
and adding them to QGIS Server would make it more competitive and attractive
to users who need these formats.

QGIS WFS/OAPIF client already supports JSON-FG and FlatGeobuf, so adding support for these formats in QGIS Server would allow to have a complete end-to-end support for these formats in QGIS.

For JSON-FG quoting https://docs.ogc.org/DRAFTS/21-045.html:

> GeoJSON is a very popular encoding for feature data. GeoJSON is widely supported,
> including in most deployments of APIs implementing the OGC API Features Standard.
> However, GeoJSON has intentional restrictions that prevent or limit its use in certain
> geospatial application contexts. For example, GeoJSON is restricted to WGS 84 coordinates,
> only supports the original Simple Features geometry types and has no concept of classifying
> features according to their type.
>
> OGC Features and Geometries JSON (JSON-FG) is a proposal for GeoJSON extensions that provide
> standard ways to support such requirements. The goal is to focus on capabilities that may
> require some geospatial expertise, but that are useful for many. Edge cases are considered
> out-of-scope of JSON-FG.
>
> Since JSON-FG specifies extensions to GeoJSON that conform to the GeoJSON standard, valid JSON-FG
> features or feature collections are also valid GeoJSON features or feature collections.


JSON-FG supports additional geometry types: CircularString, CurvePolygon, CompoundCurve, MultiCurve and
MultiSurface that are not compatible with GeoJSON RFC7946.

For FlatGeobuf quoting https://flatgeobuf.org/

> A performant binary encoding for geographic data based on flatbuffers that can hold a
> collection of Simple Features including circular interpolations as defined by SQL-MM Part 3.
> Inspired by geobuf and flatbush. Deliberately does not support random writes for simplicity
> and to be able to cluster the data on a packed Hilbert R-Tree enabling fast bounding box spatial
> filtering. The spatial index is optional to allow the format to be efficiently written as a
> stream, support appending, and for use cases where spatial filtering is not needed.
>
> Goals are to be suitable for large volumes of static data, significantly faster than legacy formats
> without size limitations for contents or metainformation and to be suitable for streaming/random access.

FlatGeobuf might offer better performance than JSON/JSON-FG, especially for large datasets,
and it is also a binary format, which can be more compact and faster to parse than JSON.


## Proposed Solution

The proposed solution is to implement support for JSON-FG and FlatGeobuf in QGIS Server OAPIF.


### JSON-FG

In order to support JSON-FG, we can leverage the existing GeoJSON support in QGIS core and extend
it to handle the additional geometry types and properties defined in the JSON-FG specification.

This means that we can reuse a lot of the existing code for GeoJSON, and only add the necessary
extensions to support JSON-FG.

In particular, we will need to add support for exporting to JSON-FG the geometry types
that are currently coerced to GeoJSON RFC7946 compatible types in QGIS core
(CircularString, CurvePolygon, CompoundCurve, MultiCurve and MultiSurface).

This will be done by implementing a new `QgsAbstractGeometry::asJsonFgObject()`
virtual method in `QgsAbstractGeometry`,  the base implementation will call `asJsonObject()`
and the geometry classes that are not JSON RFC7946 compatible will override the base method
and implement the JSON-FG specific (lossless) encoding.

At the features collection level, we will need to add support for the `featureType` property,
which is required by the JSON-FG specification to indicate the type of the features
in the collection (see the paragraph below about the feature schema).

There are also the following additional properties that we will need to support, they are described in the
JSON-FG specification, these will be added as additional properties in the JSON output by extending
`QgsJsonExporter` with `QgsJsonExporter::exportFeaturesToJsonFgObject(const QgsFeatureList &features, const QgsJsonFgExportOptions &options)`,
the options will be used to specify which profile of JSON-FG to use (RFC7946, JSON-FG or JSON-FG-PLUS) and to specify which
JSON-FG specific fields have to be exposed in the output (e.g. CRS, temporal properties, measures, etc.).

The parser for the JSON-FG specific properties will be implemented in the OAPIF handlers (it internally uses `QgsOgrUtils::stringToFeatureList()` which uses GDAL to parse the incoming JSON and should support most of the JSON-FG properties out of the box), while the options for the exporter will be set based on the profile specified in the request and on the server capabilities.

Here is the list of the additional properties that we will need to support:

#### Core classes – metadata

This lists the classes the server is conformant to, it is a collection of links to the core classes of the OGC API
Features standard, and it is used by clients to understand what capabilities the server has.

#### Core classes – temporal

This will expose temporal properties of features, such as timestamps or time intervals, which can be useful for applications that need to handle time-based data.

The existing support for time dimensions in QGIS core will be leveraged for this, and the temporal properties will be added to the JSON output as additional properties in the feature objects.

#### Core classes – CRS & Place

This will expose the CRS and the geometry in the specified CRS in addition to the default GeoJSON-compatible WGS84 geometry, which can be useful for applications that need to work with different coordinate reference systems.

####  Requirement – Circular Arcs

Support for circular arcs is a key feature of JSON-FG, and it will allow QGIS Server to support more complex
geometries that are not possible with GeoJSON. The implementation is outlined in the previous sections,
and it will involve adding JSON lossless export support for the additional geometry types defined in the JSON-FG
specification.

#### Requirement – Measures

This will allow QGIS Server to support features with measure values, which can be useful for applications
that need to handle linear referencing or other use cases that involve measures.

#### Requirement – Feature type & schemas

Quoting https://docs.ogc.org/DRAFTS/21-045.html#feature-types

> Features are often categorized by type. Typically, all features of the same type have the
> same schema and the same properties.
>
> Many GIS clients depend on knowledge about the feature type when processing feature data.
> For example, when associating a style with a feature in order to render that feature on a map display.
>
> GeoJSON is schema-less in the sense that it has no concept of feature types or feature schemas.
>
> In JSON-FG, a feature is an instance of a single feature type. If a feature is associated with
> multiple feature types, the primary type should be identified.

Feature type will be implemented at the collection (layer) level by adding an
additional string property to the server-specific layer metadata.


Quoting https://docs.ogc.org/DRAFTS/21-045.html#_overview_6

> A JSON-FG feature schema is metadata about a feature that clients can use to understand the
> content of JSON-FG features, such as a textual description of the feature properties or their value range.
>
> JSON-FG follows the approach of OGC API - Features - Part 5, that is, the feature schema is a logical schema.
> It can not be used to directly validate a JSON document. However, a schema for validation of a JSON-FG
> feature or feature collection can be derived from the logical schema, if needed. The schemas of
> the feature properties in the logical schemas can be reused when constructing a JSON Schema for validation.

See: https://portal.ogc.org/files/108199 for details.

Note that the current QGIS data model does not allow to have different feature types in the same layer, so this schema will be exposed at the collection (layer) level and all the features in the layer will share the same schema.

The schema will be automatically derived from the QGIS layer fields and properties.


#### Requirement – Profile

This is a property added to the OAPIF standard to allow specifing which profile of a certain media type
(in this case JSON-FG) is supported by the server, and it can be used by clients to understand the
capabilities of the server.

The problem comes from the REST API design, where the media type is used to specify the format of the data (content negotiation),
but it does not allow to specify different profiles of the same media type,
the solution outlined by the OAPIF standard is to use a profile property in the
query string of the request to specify which profile is requested.

Three profiles will be added to the OAPIF standard for JSON-FG:

- **rfc7946** : `http://www.opengis.net/def/profile/OGC/0/rfc7946` is the default GeoJSON profile, which is the one currently supported by QGIS server.
- **jsonfg** :`http://www.opengis.net/def/profile/OGC/0/jsonfg` JSON-FG profile
- **jsonfg-plus**: `http://www.opengis.net/def/profile/OGC/0/jsonfg-plus` is the union of the previous two profiles, it will allow to support both JSON-FG
  and GeoJSON features at the same time by providing the `geometry` member in WGS84 as a fallback for the cases where the geometry cannot be represented in GeoJSON RFC7946.

For a detailed description of the JSON-FG and JSON-FG-PLUS profiles see: https://docs.ogc.org/DRAFTS/21-045.html#place


### FlatGeobuf

In order to support FlatGeobuf, we will need to implement a new exporter class that can export features to FlatGeobuf format.

This new class will internally use GDAL to export the features to FlatGeobuf format, leveraging the existing GDAL support for FlatGeobuf.

The API of the new class is not defined yet, but it will be similar to the existing `QgsJsonExporter` class, with a method like `exportFeaturesToFlatGeobuf(const QgsFeatureList &features, const QgsFlatGeobufExportOptions &options)`.

Note that being FlatGeobuf a binary format, the output will be a byte array instead of a JSON object and the next/prev links in the (paged) OAPIF response will need to be sent in the headers instead of the body of the response, this imply writing a dedicated method to handle the OAPIF response for FlatGeobuf output, in order to set the appropriate headers and to write the binary output to the response body (this will not have any impact on the public API).

TODO: support for range requests will be considered for future improvements, but it is not in the scope of the current implementation.

## Deliverables

- Implementation of JSON-FG and JSON-FG-PLUS support in QGIS Server OAPIF, including support for the additional geometry types, properties and profiles defined in the JSON-FG specification.
- Implementation of FlatGeobuf support in QGIS Server OAPIF, including a new exporter class that can export features to FlatGeobuf format.
- Documentation of the new features and capabilities added to QGIS Server OAPIF, including examples of how to use the new output formats.
- Tests to ensure the correctness and performance of the new features, including unit tests for the new exporter classes and integration tests for the OAPIF endpoints that support the new output formats.

### Example(s)

None.

### Affected Files

#### Core

- `qgsabstractgeometry.cpp/h`: for the new `asJsonFgObject()` virtual method and the implementation of the JSON-FG specific encoding for the additional geometry types.
- various geometry classes (e.g. `qgscircularstring.cpp/h`, `qgscurvepolygon.cpp/h`, etc.): for the implementation of the JSON-FG specific encoding `asJsonFgObject()` for the additional geometry types.
- `qgsjsonutils.cpp/h`: for the JSON-FG export support, we will need to add new methods to handle the additional geometry types and properties defined in the JSON-FG specification.
- `qgsjsonexpoerter.cpp/h` for `QgsJsonExporter::exportFeaturesToJsonFgObject()`

#### Server


- `qgswfs3handler.cpp` for the implementation of the OAPIF endpoints that support the new output formats, we will need to add new methods to handle the requests for JSON-FG and FlatGeobuf formats, and to use the new exporter classes to generate the output.

In particular, the handlers will need to inspect the new profile property in the request to determine which profile of JSON-FG to use for the output, and to set the appropriate options for the exporter classes.

An endpoint for the features schema will also be added, to allow clients to retrieve the schema of the features in a collection (layer) in JSON-FG format.

New HTML template for the schema endpoint will be added to the server templates, to allow clients to retrieve the schema of the features in a collection (layer) in a human-readable format (HTML).

## Risks

The implementation will not have an impact on the existing formats, which will continue to work as before.

There are some risks related to the complexity of the JSON-FG specification, which has many optional properties and profiles, and it might be challenging to implement all the features and to ensure that they work correctly together.

## Performance Implications

There are no expected performance implications for the existing formats
because the new output formats will use different (or extended) code paths.

The additional FlatGeobuf format might offer better performance than JSON/JSON-FG, especially for large datasets. However, the actual performance gain
will depend on the specific use case.

## Further Considerations/Improvements

"Building blocks for 3D geometries" is an optional conformance class and it will not
be supported in the current implementation, but it can be considered for future improvements if there is enough demand for it.

See:
https://docs.ogc.org/DRAFTS/21-045.html#bb_3d

Range requests for FlatGeobuf will not be supported in the current implementation, but it can be considered for future improvements if there is enough demand for it.

## Backwards Compatibility

None.

## Issue Tracking ID(s)

None.


## Funding

This work has already secured a small kickoff funding from the QGIS-DE - Anwendergruppe Deutschland e.V. and there is already a proof of concept working implementation for FlatGeobuf, but additional funding will be needed to complete the implementation and to write the tests and documentation.

