# QGIS Enhancement: Title

**Date** 2026/01/06

**Author** Sandro Santilli (@strk)

**Contact** strk@kbt.io

**Version** QGIS 4.X

# Summary

Port the DBManager TopoViewer plugin to the data browser

## Motivation

There have been a long standing will to remove the DBManager from core,
and the TopoViewer plugin is one of the things that would be missing.

## Proposed Solution

Each PostGIS Topology schema shown in the data browser would get an additional
item in the "Schema Operations" right-click menu: "TopoViewer".

Clicking on the "TopoViewer" would load a tree of layers with the same
structure of the one currently used by the DBManager TopoViewer plugin.

### Affected Files

TBD

## Risks

None anticipated

## Performance Implications

None anticipated

## Further Considerations/Improvements

In the future the templates could be made tweakable
