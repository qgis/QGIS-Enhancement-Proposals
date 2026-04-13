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



## Proposed Solution
We propose to go through all processing algorithms, identify the parameters' role and include a translatable string as a help message for all parameters that do not have one.

We also suggest to add the help string as an optional parameter in all `QgsProcessingParameterDefinition` subclasses constructors. This will allow for creating a CI test to force using help strings for all parameters, without breaking the API for the Processing Parameter subclasses constructors.

## Deliverables

- Modified constructors for `QgsProcessingParameterDefinition` subclasses
- Updated `initAlgorithm()` methods in processing algorithms to include the parameters' new help string
- CI test that will fail when processing parameters without a help string are created.

### Affected Files

- `src/core/processing/qgsprocessingparameters.[cpp|h]`
- potentially constructors of other `QgsProcessingParameterDefinition` subclasses in seperate files
`initAlgorithm` methods in:
- `src/analysis/processing/*.cpp
- `python/plugins/processing/algs/gdal/*.py`
- `python/plugins/processing/algs/qgis/*.py`

## Risks

- A bulk of new strings for translation will be introduced, adding extra work to the translators.
- We may end up with redundant help messages for self explanatory parameters

## Performance Implications

N/A

## Further Considerations/Improvements
In QGIS 5 we can consider changing the API to require both `description` and `help` for all `QgsProcessingParameterDefinition` subclasses.


## Backwards Compatibility

N/A
