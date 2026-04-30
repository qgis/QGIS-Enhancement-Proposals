# QGIS Enhancement: Force usage of help strings for processing algorithm parameters

**Date** 2026/04/13

**Author** Stefanos Natsis (@uclaros)

**Contact** uclaros at gmail dot com

**Version** QGIS 4.4

# Summary

The labels of processing algorithm parameters are not always self explenatory. In the past, I have resorted to reading the source code in order to understand what a parameter does.
While the algorithms' help messages may include descriptions for the parameters, it is often inadequate or may even be overlooked by the users.

Tooltips are arguably the most accessible way to communicate to users some extra information about those parameters.
In QGIS 3.16 custom help messages for parameters displayed on tooltips was introduced. These get defined using `QgsProcessingParameterDefinition::setHelp( const QString &help )` for each instantiated parameter.
Many new algorithms use the `setHelp()` method for their parameters, offering a nice UX to users, however its usage was not globally adopted, resulting in lots of parameters with no description.

On the other hand, the QGIS documentation pages have a rich collection of processing algorithm parameter descriptions.


## Proposed Solution
We propose to go through all processing algorithms and transfer the parameters' descriptions from the QGIS documentation pages into `setHelp()` calls to make them available in tooltips.
Some descriptions may require modification to fit the context of the processing algorithm tooltips, eg. for combo boxes the description of their funtion is required but the available values are not.

We also suggest to add a CI test to force using help strings for all parameters, excluding input and output ones. This will ensure that new algorithms have proper descriptions for their parameters which the documenters will be able to reuse for the documentation repo.

## Deliverables

- Updated `initAlgorithm()` methods in processing algorithms to include the parameters' new help strings
- CI test that will fail when processing parameters without a help string are created.

### Affected Files

- `src/analysis/processing/*.cpp
- `python/plugins/processing/algs/gdal/*.py`
- `python/plugins/processing/algs/qgis/*.py`
- `python/plugins/processing/tests/AlgorithmsTestBase.py`

## Risks

- A bulk of new strings for translation will be introduced, adding extra work to the translators.
- We may end up with redundant help messages for self explanatory parameters

## Performance Implications

N/A

## Further Considerations/Improvements

For parameters that lack a proper description, a new translatable string will be added and pushed to the documentation repo, too.


## Backwards Compatibility

N/A
