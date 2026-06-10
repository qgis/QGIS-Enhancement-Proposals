# QGIS Enhancement 426: Demotion of DB Manager plugin to external plugin

## Summary

This QEP proposes the removal of the DB Manager plugin from the core QGIS repository (qgis/qgis) and
the default QGIS installations.

## Justification

Historically, the DB Manager plugin was a critical component of the QGIS ecosystem, providing essential
database administration and querying capabilities. However, over the 3.x development cycle, the 
direction of QGIS shifted toward integrating all database related functionality into the Browser Panel. This
provides a better user experience, by exposing database tools alongside other layer and connection management
tools. The browser-based functionality is all designed around generic, heavily tested connection APIs, which
are also used by many other areas of QGIS. In contrast, the DB Manager plugin contains all its own logic
and code for handling database integration, with extremely minimal (almost non-existent) test coverage.

As of QGIS 4.2, the core functionality of the DB Manager has been fully ported to the built-in
Browser Panel. Users can natively manage schemas, create and delete tables, manage fields, and execute
SQL queries directly within the core interface. Maintaining the DB Manager plugin as a core component now
duplicates this functionality, bloating the codebase and creating unnecessary maintenance overhead
for core developers.

While the DB Manager does contain some features that have not been ported to the Browser Panel, these
remaining features are niche functionality. Please see https://docs.google.com/spreadsheets/d/1VyC_kYJU3qmWrzXzjeZuHGBjwE6O2lPov_c0LnO-AUA/edit?gid=0#gid=0
for a detailed write up of the remaining unported functionality (thanks to Alexandre Neto!).

Users who rely on this unported functionality have two paths forward:

1. Sponsorship: Organizations and users who depend on specific missing features are heavily encouraged to sponsor
   core developers to port those capabilities directly into the Browser Panel.
2. External Maintenance: Because the DB Manager will remain available as a standalone plugin, a motivated
   community member or organization can step up to maintain and extend it outside of the core QGIS release cycle.

## Proposed Implementation Plan

### Code Extraction and Archival

The DB Manager code will be completely removed from the qgis/qgis repository. The removed code will be migrated to
a new, dedicated repository at qgis/db_manager_plugin.

In order to reflect its unmaintained state, this new repository will be explicitly set to read-only, and the
issue tracker will be disabled.

A clear note will be added to the README.md of the new repository describing the plugin's unmaintained status
and putting out a call for volunteer maintainers.

### Final Plugin Version Publication

A final release representing the current state of the plugin will be tagged in the new repository. This version
will be packaged and published to the official QGIS plugin repository.

This ensures that users upgrading their QGIS version can still download and install the plugin if they 
need its legacy functionality.

### Community Handoff

Following the initial publication, the plugin will officially be considered unmaintained. No further releases,
bug fixes, or compatibility updates will be planned or executed by the core QGIS team.

A "call for volunteer maintainers" will be put out via the QGIS-dev, QGIS-user mailing list, and via the QGIS
news feed. This will call for a motivated community member or organization to fork the plugin and volunteer
as its new maintainer.

Once a community member has forked the plugin and established themselves as the new maintainer, the
transitional qgis/db_manager_plugin repository will be permanently dropped/deleted.

## Timeline

It is proposed that the demotion occurs for the QGIS 4.2 release, to coincide with the first
LTR release in the 4.x cycle

