# QGIS enhancement: Get rid of `QgsProject::instance()` singleton in qgis_core

**Date** 2026-04-13

**Author** Martin Dobias (@wonder-sk)

**Contact** wonder dot sk at gmail dot com

**Version** QGIS 4.4

## Summary

`QgsProject::instance()` is a global singleton that returns the current project. It is convenient within the QGIS app - there is only one project loaded at a time. But for code in `qgis_core` (and to a lesser extent in providers) the singleton is a poor fit:

- `qgis_core` is a library. It can be used in contexts where multiple `QgsProject` instances exist at once - e.g. in Python scripts, 3rd party apps, tests.
- API user can get unexpected results when some API method somewhere deep inside qgis_core code uses QgsProject singleton, without the user knowing about it. That may block reasonable uses or introduce subtle bugs.

The goal of this proposal is to remove all uses of `QgsProject::instance()` from `qgis_core`, by explicitly passing the relevant `QgsProject` object (or a smaller object - e.g. `QgsCoordinateTransformContext`, `QgsExpressionContext`).

## Prior Work

This is not a new idea - the work started in 2016 during 3.0 release cycle within QgsProject refactoring (see https://github.com/qgis/QGIS/pull/3855). There were multiple follow ups in early 2017, but then the work mostly stalled. There are sill around 100 occurrences of `QgsProject::instance()` in `src/core` and ~40 in `src/providers` as of 4.0.

One more important step happened in 2024: Nyall added a banned-keyword check to the pre-commit hook that rejects any new `QgsProject::instance()` introduced under `src/core` (https://github.com/qgis/QGIS/pull/59118). All existing call sites at that point were marked with a `// skip-keyword-check` trailing comment. That effectively stopped addition of new occurrences.

This QEP is essentially continuation of the original 2016-17 project refactoring with a clear target to get rid of the `// skip-keyword-check` list in qgis_core.

## Proposed Solution

With roughly 100 occurrences of `QgsProject::instance()`, there will be several categories of fixes:

1. "Mechanical" fixes - these are the cases where QgsProject is already reachable directly, or nearly reachable, and just needs to be propagated from callers. In various cases the solution will be to use a "smaller" context object rather than "full" QgsProject.

2. Classes that depend on QgsProject pretty much for their lifetime, but they are not getting it in the constructor (e.g. QgsOfflineEditing). These should most often get a new constructor with QgsProject, and get the old one deprecated in the API. Still mostly "mechanical" fixes, even though more involved.

3. Harder "architectural" fixes - these are places where some design decisions will need to be made. For example, QgsSymbol::defaultSymbol() static method that uses QgsProject singleton, but there's no project in the context.

Why not just delete `QgsProject::instance()`? Two main reasons why not:

1. stability - it is in public API, and heavily used - we don't want to break API
2. practical - there are hundreds of cases in QGIS application and getting rid of all of them would be a lot of work. At the same time, there are no plans to support multiple "current" projects within QGIS desktop app.


## Deliverables

The deliverables will be a series of pull requests, lowering number of occurences of QgsProject::instance() with each. The target is to get to zero occurences within qgis_core library, but due to the lack of time a couple of hard-to-fix cases may be left for future fixes. If the fixes went exceptionally well and time will allow, src/providers/ may be the next part of code to look into.

## Risks

As mentioned above, some of the cases may be hard to fix and may need to be omitted if the time is tight.

## Performance Implications

None

## Backwards Compatibility

None of the changes will break API.
