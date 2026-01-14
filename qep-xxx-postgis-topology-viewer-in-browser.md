# QGIS Enhancement: Title

**Date** 2026/01/14

**Author** Sandro Santilli (@strk)

**Contact** strk@kbt.io

**Version** QGIS 4.0

# Summary

Port the DBManager TopoViewer plugin to the the QGIS browser.

## Motivation

There's some desire to remove the DBManager from core, making the functionality available via C++ code.

Reference: https://lists.osgeo.org/pipermail/qgis-developer/2022-June/064850.html

The TopoViewer plugin is one of the feature that's currently only available in the DBManager plugin.

## Proposed Solution

Port the TopoViewer functionality from DBManager to Browser.

## Deliverables

Ability to load a whole PostGIS Topology layer group from a catalog of topologies found in each PosGIS database supporting them.

## Risks

None
