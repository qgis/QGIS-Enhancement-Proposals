# QGIS Enhancement 427: Removal of deprecated Processing GUI API

## Summary

This QEP proposes that the deprecated Processing ["WidgetWrapper" API](https://github.com/qgis/QGIS/blob/master/python/plugins/processing/gui/wrappers.py)
(from QGIS 2.x) be removed, requiring all plugins and third party code to use the replacement [QgsAbstractProcessingParameterWidgetWrapper](https://qgis.org/pyqgis/master/gui/QgsAbstractProcessingParameterWidgetWrapper.html) API
instead (which was introduced in QGIS 3.4).

## Justification

As per clause 1.4.4 of the Stable API policy, this API should be removed on the grounds that
maintaining backward compatibility both introduces an insurmountable maintenance burden and fundamentally
blocks critical bug fixes (condition 1.4.4.1).

This pure Python API is an ongoing source of unsolvable issues in the Processing framework, such as
https://github.com/qgis/QGIS/issues/66282. It relies on a fragile memory management model, where the
lifetime of objects created in C++ vs Python results in unpredictable object deletion or memory leakage.
Over time the conclusion from QGIS developers familiar with this code is that this approach just
cannot be fixed, and we need to move solely to the approach used elsewhere in QGIS, where Python is used
**solely** to subclass C++ base classes, with all memory management living at the C++ level (either through
direct C++ memory management or via C++ Qt parent/child associations).

In addition, the old API predated major overhauls of Processing GUI components such as the model designer,
and the assumptions used in that API no longer apply (such as existence of Python dialog classes). Working
around these old assumptions is a considerable drain on QGIS developers, and is blocking further desirable
revamps and modernisation of the QGIS model designer interface.

As per clause 1.4.4.3, a "deprecation warning should be provided to users for at least one minor release
before the breakage occurs". In the case of this API, we have provided a deprecation warning in advance 
since QGIS 3.8 (over 7 years ago!) (see [8c4cbfca0](https://github.com/qgis/QGIS/commit/234c1718c4cbfca0867cb6a2bdabdc7524e17a7f).

## Proposed Implementation Plan

### 1. Port remaining QGIS algorithm widgets to new API

There remain a handful of parameter widgets using the old API from the inbuilt QGIS
algorithms (see https://github.com/qgis/QGIS/tree/master/python/plugins/processing/algs/qgis/ui). Specifically,
these are:

- The Execute SQL widget
- The interpolation algorithm custom widgets
- Raster calculator custom widgets

These widgets will need porting to the new API before the deprecated API can be removed. There are no
technical reasons why this has not been done to date -- it's purely that there was no pressing motivation
for them to be ported yet.

It is proposed that these widgets be ported to C++ at the same time as they are ported to the replacement
API, and their associated algorithms also ported to C++. Additionally, regression tests of the custom
widget logic should be introduced at this same time (none exist for the existing implementations).

Ideally, this porting would happen during the QGIS 4.4 release cycle.

### 2. Remove unused WidgetWrapper subclasses

[wrappers.py](https://github.com/qgis/QGIS/blob/master/python/plugins/processing/gui/wrappers.py) contains
many subclasses of the deprecated `WidgetWrapper` class, such as  `CrsWidgetWrapper`, `PointWidgetWrapper`, etc.
These are unused in current master code, and have been retained only to maintain stable API in the case
that a third party plugin is using them for their own algorithm parameters.

These subclasses can be removed immediately with no impact on any other code in the master version of QGIS.

This will happen during the QGIS 4.4 release cycle.

### 3. Removal of WidgetWrapper class

Only when both tasks (1) and (2) are completed can the `WidgetWrapper` class itself can be safely removed.
The logic in `WidgetWrapperFactory` will be simplified so that only the factory logic using the
C++ `QgsGui.processingGuiRegistry()` methods will be retained.

The time frame for this depends on completion of tasks (1) and (2), but ideally will occur during
the 4.4 release cycle.

### 4. Removal of Python ParametersPanel class

Most of the logic in the `ParametersPanel` class now lives in the underlying C++ `QgsProcessingParametersWidget`
class. The remaining logic relates solely to the deprecated `WidgetWrapper` and `WidgetWrapperFactory` API.

When task (3) is complete we will be able to move the remaining logic from `ParametersPanel` into the
underlying `QgsProcessingParametersWidget` class, and drop the `ParametersPanel` class entirely.

The GDAL provider `GdalAlgorithmWidget` will also need updating accordingly.

The time frame for this depends on task (3), but realistically will likely occur no sooner than the 4.6
release cycle.

### 5. Removal of Python AlgorithmWidget class

Much like the parameters panel class, the `AlgorithmWidget` is a Python subclass of an underlying C++
`QgsProcessingAlgorithmWidgetBase` class. When task (4) is removed and there remains no use left of
`ParametersPanel` in `AlgorithmWidget`, we will be free to move the remaining logic from the Python
subclass into the underlying `QgsProcessingAlgorithmWidgetBase` class.

This task is considerably more involved than task (4), as the `AlgorithmWidget` Python subclass
currently contains ALL the logic for triggering the algorithm execution and handling post-processing
tasks after the algorithm completes. Accordingly the code from the Python [handleAlgorithmResults](https://github.com/qgis/QGIS/blob/master/python/plugins/processing/gui/Postprocessing.py#L119) method
will also need to be ported to C++, or an interface added to the C++ Processing GUI utility
classes so that the Python logic can be called from C++ widget code. (Note that this existing post-processing
logic is fragile and is a frequent cause of issues and regression in Processing!).

The existing Python-based model designer and batch execution dialogs will need to be tweaked to remove
all references to the removed `AlgorithmWidget` class, and to rely solely on the C++ classes alone.

It is considered out-of-scope for this project to completely port the Model Designer or Batch dialog to
C++, however, these classes are NOT part of the stable API and this work could be completed at anytime
following the completion of this project.

The time frame for this depends on task (4), but realistically will likely occur no sooner than the 4.6
release cycle.
