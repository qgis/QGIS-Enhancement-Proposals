# QGIS Enhancement: Expression filter on vector layers

**Date** 2026/03/29

**Author** Dave Signer (@signedav)

**Contact** david at opengis dot ch

**Version** QGIS 4.2

# Summary

An expression filter similar to the query builder, accessible via *Layer > Filter*, where the entire layer can be filtered using expressions. You can choose between one of the filters. This enables us to filter all types of vector layers according to QGIS expressions and use elements such as variables, custom functions and more.

# Proposed Solution

## UX

You will be able to choose the filter type: "WHERE clause" or "QGIS Expression", while the "WHERE clause" can have additional info considering `QgsDataProvider::subsetStringDialect()`.

When "WHERE clause" is chosen, you will see the "traditional" query builder content (Mockup):

![querybuilder-gui](images/qep416/querybuilder-gui.png)

When "QGIS Expression" is chosen, you will see the expression builder where you can create your expression (Mockup):

![expression-gui](images/qep416/expression-gui.png)

This would include at global scope and project scope. But we must be aware that with this the provider will need access to the project. Additionally, we need to be careful when accessing the project from a background thread.

We won't offer the combination of both. Even though it would be technically possible, it seems fairly fault-prone and hard to use. Making the choice between provider side filter or QGIS expression filter mutually exclusive will lead to a more user-friendly experience. (If a user wants to combine a provider side filter with qgis expression filtering, they have other options like setting a rule-based renderer on the layer to apply a qgis expression filter at render time only.)

## Backend

### Set the subset expression

This subset expression will be set via the layer `QgsVectorLayer::setSubsetString()` and provider `QgsDataProvider::setSubsetString()`, landing in the `QgsAbstractFeatureSource` as the member `mSubsetString`. To divert it to the existing SubsetString new enumeration is introduced `ProviderSubsetFilterType` (`Query`|`Expression`).

It is stored as the current custom property `QgsMapLayer::mCustomProperties['storedSubsetString']` and `QgsMapLayer::mCustomProperties['providerSubsetFilterType']`.

In the feature iterator, we then merge it into the feature request's filter expression so they can be compiled together into a server side filter (completely or partially) if possible. Otherwise, they are evaluated together on the client side.
Because this must be implemented individually within every provider iterator, not all providers will support it from the beginning. To indicate whether subset expressions are supported it will has a flag in the provider's capabilities metadata.

### Merging to the filter expression

The idea is to merge the `subsetExpression` with the existing `filterExpression` in the feature request.

This must be done in the constructor of the feature iterator's derivative. For example:

```c++
QgsPostgresFeatureIterator::QgsPostgresFeatureIterator( QgsPostgresFeatureSource *source, bool ownSource, const QgsFeatureRequest &request )
  : QgsAbstractFeatureIteratorFromSource<QgsPostgresFeatureSource>( source, ownSource, request )
{
  
  [...]

  request.combineFilterExpression(source.subsetExpression(), true);

  [...]

```

`combineFilterExpression` merges the expression with the existing filter epression using `AND`. The `combineFilterExpression` should be extended by a parameter `bool outer = false` which we set to `true` to place the `subsetExpression` before the `filterExpression` (becoming the outer filter). This is due to short-circuit evaluation (returning directly when the first expression is `false`) and the assumption that the `subsetExpression` is generally less complex than the `filterExpression`. Of course, this is not always the case. Perhaps a cost-calculator would make sense, but this is not part of this enhancement.

Also, it needs to set the filter type to `Expression` because the filter expression will not be evaluated on the client side otherwise.

```c++
  request.setFilterType( Qgis::FeatureRequestFilterType::Expression);
``` 

(`setFilterType` does not exist yet)

### Merging to the fids

There is also a way to filter according to feature IDs: `QgsFeatureRequest( QgsFeatureId fid )` and `QgsFeatureRequest( const QgsFeatureIds &fids )`.  This needs to be upgraded to a filter expression type.

##### When providers can compile

If the providers are able to compile expressions into server side queries, this can be done with feature IDs as well. For example, in `QgsPostgresFeatureIterator`, the IDs are added to the `whereClause` regardless (this is existing code):

```c++
  if ( request.filterType() == Qgis::FeatureRequestFilterType::Fid )
  {
    QString fidWhereClause = QgsPostgresUtils::whereClause( mRequest.filterFid(), mSource->mFields, mConn, mSource->mPrimaryKeyType, mSource->mPrimaryKeyAttrs, mSource->mShared );

    whereClause = QgsPostgresUtils::andWhereClauses( whereClause, fidWhereClause );
  }

```

Technically, the functionality remains the same with a combined filter expression. But instead of appending it to the `whereClause`, it is added to the filter expression using the specific filter expression type.

#### When providers cannot compile

When the feature ID where-clause cannot be passed to the server, we must append it to the filter expression as well:

```c++
QgsAnotherIterator::QgsAnotherIterator( QgsAnotherIterator *source, bool ownSource, const QgsFeatureRequest &request )
  : QgsAbstractFeatureIteratorFromSource<QgsPostgresFeatureSource>( source, ownSource, request )
{
  
  [...]

  if ( request.filterType() == Qgis::FeatureRequestFilterType::Fid )
  {
    //create expression string
    const QString fidExpression = u"@id=%1"_s.arg( mRequest.filterFid() );

    //combine it to the filter expression
    request.combineFilterExpression(fidExpression, true);

    //upgrade filter type
    request.setFilterType( Qgis::FeatureRequestFilterType::Expression);
  }
  [...]
```

### When having no feature request filter yet

On `Qgis::FeatureRequestFilterType::NoFilter`, the `subsetExpression` has to be added to the (empty) `filterExpression` and the filter type needs to be upgraded like above as well.

### Additional places to consider

#### Extent 

For example, via "Zoom to Layer(s)".

This can be integrated once we are able to compile the subset expression. In PostgreSQL e.g. in `computeExtend3D` by passing the cumpiled subset expression instead of the `filterWhereClause`.

If this is not possible, we would need to iterate through all the fetched features, which can be a performance killer.

That is why we suggest calculating the extent only on the server side. When there is a partial compilation of the subset expression, the extent considers a superset of the actual filtered features (meaning it can be larger).

#### Feature Count

For example, the feature count in the layer tree.

This can be integrated once we are able to compile the subset expression as part of the filter.

If this is not possible, we would need to iterate through all the fetched features, which can be a performance killer.

That is why we suggest not calculating any feature count on the client side and showing the count in the layer tree as `[N/A]`.

#### Optional preview in the GUI

Visual feedback would be very helpful in the GUI to know whether the expression can be *compiled* or not. But this is not part of this enhancement.

### Affected Files

- qgsvectorlayer.h / cpp to handle new providerSubsetFilterType etc. and probably featureCount/extent
- qgsdataprovider.h / cpp to handle new providerSubsetFilterType etc. and probably featureCount/extent
- qgsfeaturerequest.cpp to improve `combineFilterExpression` and set providerSubsetFilterType in the `QgsAbstractFeatureSource`
- Provider-specific feature iterators (like e.g. qgspostgresfeatureiterator.cpp)
- Provider-specific test suites

## Deliverables

- Expression subsets on vector layers based on PostgreSQL / OGR
- Improving PostgreSQL iterator handling of partially compiled expressions (because it currently only supports complete compilation or none at all)

## Non-Deliverables

The following are not part of this QEP:

- Expression subsets on all the other vector layers
- Optional preview in the GUI indicating whether it can be compiled (see above)
- Cost calculator to evaluate if the part should be an outer or inner filter

## Risks

None

## Performance Implications

Avoided by not providing full support on client side extend / featureCount

## Further Considerations/Improvements

See the non-delivered parts above.

## Backwards Compatibility

Stored subset expressions will simply be ignored in older QGIS versions.