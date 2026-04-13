# QGIS Enhancement: Async Refactoring

**Date** 2026/04/13

**Author** David Koňařík (@dvdkon)

**Contact** dvdkon at konarici dot cz

**Version** QGIS 4.4

# Summary

QGIS does a lot of operations asynchronously, to not block the main thread and
to improve throughput. Currently, those operations are handled by low-level
primitives, like threads, signals and at best a `QFutureWatcher`. This approach
is complex and bug-prone, as can be seen in e.g. [this 3D tile loading
issue](https://github.com/qgis/QGIS/issues/63340).

In Qt6, better and more modern primitives for writing asynchronous code [were
added](https://www.qt.io/blog/asynchronous-apis-in-qt-6), like `QPromise` and
new methods on `QFuture`. Having upgraded to Qt6, we can now use these to
refactor our async code using paradigms proven by use in other languages and
frameworks.

## Proposed Solution

The proposed refactoring will replace uses of raw signals and `QFutureWatcher`
by `QPromise`s and chained `QFuture`s wherever applicable.

## Deliverables

The main deliverable is refactoring the 3D chunk loading code (`QgsChunkLoader`
and associated classes) according to the principles outlined above. If funds
are left over, other code might also be refactored, such as:

- `QgsMesh3DGeometryBuilder`
- `QgsGeometryChecker`
- `QgsProfilePlotRenderer`

## Risks

The only known risk here is that this change in style might not be widely
accepted by QGIS developers. The resulting style change should therefore be
accepted as part of this QEP.

## Performance Implications

None known.

## Further Considerations/Improvements

Using the [new Qt Task
Tree](https://www.qt.io/blog/qt-task-tree-coming-soon-in-qt-6.11) was
considered, but it seems not to fit the needs of QGIS in modernising async
primitives. Using `QPromise`/`QFuture` directly has the added benefit of their
model of operation being widely known from other programming languages,
including Python.

Using [C++
coroutines](https://en.cppreference.com/w/cpp/language/coroutines.html) with
`QFuture`s (as some are [already
doing](https://www.arnorehn.de/blog/2026/02/09/qfuture-c-coroutines/)) might
further improve the developer experience of handling async code, but it still
seems early to commit to using such a feature in a large and conservative
project like QGIS.

## Backwards Compatibility

If any code in public methods will be touched, new methods will be added which
use the new paradigm (e.g. returning a `QFuture` instead of returning `void`
and later emitting a signal), with the old methods made into wrappers.
